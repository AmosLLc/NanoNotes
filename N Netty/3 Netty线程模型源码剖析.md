[TOC]

### Netty线程模型源码剖析

#### 为什么要看源码

1、**提升技术功底：**学习源码里的优秀设计思想，比如一些疑难问题的解决思路，还有一些优秀的设计模式，整体提升自己的技术功底

2、**深度掌握技术框架：**源码看多了，对于一个新技术或框架的掌握速度会有大幅提升，看下框架demo大致就能知道底层的实现，技术框架更新再快也不怕

3、**快速定位线上问题：**遇到线上问题，特别是框架源码里的问题(比如bug)，能够快速定位，这就是相比其他没看过源码的人的优势

4、**对面试大有裨益：**面试一线互联网公司对于框架技术一般都会问到源码级别的实现

5、**知其然知其所以然：**对技术有追求的人必做之事，使用了一个好的框架，很想知道底层是如何实现的

6、**拥抱开源社区：**参与到开源项目的研发，结识更多大牛，积累更多优质人脉

#### **看源码方法(**凭经验去猜)

1、**先使用：**先看官方文档快速掌握框架的**基本使用**。

2、**抓主线：**找一个 demo 入手，顺藤摸瓜快速**静态**看一遍框架的**主线**源码(**抓大放小**)，**画出源码主流程图**，切勿一开始就**陷入源码的细枝末节**，否则会把自己绕晕。

3、**画图做笔记：**总结框架的一些**核心**功能点，从这些功能点入手深入到源码的细节，**边看源码边画源码走向图**，并对关键**源码的理解做笔记**，把**源码里的闪光点都记录下来**，后续借鉴到工作项目中，理解能力强的可以直接看静态源码，也可以边看源码边 debug 源码执行过程，观察一些关键变量的值。

4、**整合总结：**所有功能点的源码都分析完后，回到主流程图再梳理一遍，争取把自己画的所有图都在脑袋里做一个整合。

#### Netty线程模型图

![Netty线程模型图](assets/85408)

#### Netty线程模型源码剖析图

图链接：https://www.processon.com/view/link/5dee0943e4b079080a26c2ac

![img](assets/85410)



Netty 就是对 Java NIO 代码的进一步封装。

#### Netty高并发高性能架构设计精髓

- **主从 Reactor 线程模型**
- NIO **多路复用**非阻塞
- **无锁串行化**设计思想
- 支持**高性能序列化**协议
- **零拷贝**(直接内存的使用)
- ByteBuf **内存池**设计
- 灵活的 TCP **参数配置**能力
- 并发优化

##### 1. 无锁串行化设计思想

在大多数场景下，并行多线程处理可以提升系统的并发性能。但是，如果对于共享资源的并发访问处理不当，会带来严重的锁竞争，这最终会导致性能的下降。为了尽可能的避免锁竞争带来的性能损耗，可以通过串行化设计，即消息的处理尽可能在同一个线程内完成，期间不进行线程切换，这样就避免了多线程竞争和同步锁。

为了尽可能提升性能，Netty 采用了串行无锁化设计，在 IO 线程内部进行串行操作，避免多线程竞争导致的性能下降。表面上看，串行化设计似乎 CPU 利用率不高，并发程度不够。但是，通过调整 NIO 线程池的线程参数，可以同时启动多个串行化的线程并行运行，这种局部无锁化的串行线程设计相比一个队列-多个工作线程模型性能更优。

Netty 的 NioEventLoop 读取到消息之后，直接调用 ChannelPipeline 的 fireChannelRead(Object msg)，只要用户不主动切换线程，一直会由 NioEventLoop 调用到用户的 Handler，期间不进行线程切换，这种串行化处理方式避免了多线程操作导致的锁的竞争，从性能角度看是最优的。

一个 Pipeline 中有多个 Handler 也会在一个线程内全部串行执行，不涉及线程切换等问题。

##### 2. ByteBuf内存池设计

随着 JVM 虚拟机和 JIT 即时编译技术的发展，对象的分配和回收是个非常轻量级的工作。但是对于缓冲区 Buffer (相当于一个内存块)，情况却稍有不同，特别是对于堆外直接内存的分配和回收，是一件耗时的操作。为了尽量重用缓冲区， Netty 提供了基于 ByteBuf 内存池的缓冲区重用机制。需要的时候直接从池子里获取 ByteBuf 使用即可，使用完毕之后就重新放回到池子里去。下面我们一起看下 Netty ByteBuf 的实现：

<img src="assets/85205" alt="img" style="zoom:70%;" />

 

可以看下 netty 的读写源码里面用到的 ByteBuf 内存池，比如 read 源码 **NioByteUnsafe**.**read**()

<img src="assets/85212" alt="img" style="zoom:60%;" />

<img src="assets/85213" alt="img" style="zoom:67%;" />

<img src="assets/85221" alt="img" style="zoom:67%;" />

继续看 newDirectBuffer 方法，我们发现它是一个**抽象方法**，由 AbstractByteBufAllocator 的子类负责具体实现，代码如下：

<img src="assets/85225" alt="img" style="zoom:67%;" />

代码跳转到 PooledByteBufAllocator 的 newDirectBuffer 方法，从 Cache 中获取内存区域 PoolArena，调用它的 **allocate** 方法进行**内存分配**：

![img](assets/85227)

 PoolArena 的 allocate 方法如下：

![img](assets/85229)

 我们重点分析 newByteBuf 的实现，它同样是个抽象方法，由子类 DirectArena 和 HeapArena 来实现不同类型的**缓冲区分配**

![img](assets/85231)

 我们这里使用的是直接内存，因此重点分析 DirectArena 的实现

![img](assets/85235)

 最终执行了 PooledUnsafeDirectByteBuf 的 **newInstance** 方法，代码如下：

<img src="assets/85240" alt="img" style="zoom:67%;" />

通过 RECYCLER 的 get 方法循环使用 ByteBu f对象，如果是非内存池实现，则直接创建一个**新的 ByteBuf 对象**。

##### 3. 灵活的TCP参数配置能力

合理设置 TCP 参数在某些场景下对于性能的提升可以起到显著的效果，例如接收缓冲区 SO_RCVBUF 和发送缓冲区 SO_SNDBUF。如果设置不当，对性能的影响是非常大的。通常建议值为 128K 或者 256K。

Netty 在启动辅助类 **ChannelOption** 中可以灵活的**配置 TCP 参数**，满足不同的用户场景。

<img src="assets/85175" alt="img" style="zoom:70%;" />

##### 4. 并发优化

- **volatile** 的大量、正确使用;
- **CAS** 和原子类的广泛使用；
- 线程**安全容器**的使用；
- 通过**读写锁**提升并发性能。