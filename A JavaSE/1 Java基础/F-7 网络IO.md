[TOC]

### 网络IO

#### 基本概念

##### 1. 同步与异步

同步和异步是针对应用**程序和内核的交互**而言的。

##### 2. 阻塞与非阻塞

阻塞和非阻塞是针对于**进程在访问数据**的时候，根据 IO 操作的**就绪状态**来采取的不同方式，说白了是一种读取或者写入操作函数的实现方式，阻塞方式下读取或者写入函数将一直等待，而非阻塞方式下，读取或者写入函数会立即返回一个状态值。

**由上描述基本可以总结一句简短的话，同步和异步是目的，阻塞和非阻塞是实现方式。**



#### BIO

BIO（Blocking IO） 是**面向流**的**同步阻塞 IO**。Java BIO 面向流意味着每次从流中读**一个或多个字节**，**直到读取所有字节**，**没有被缓存**到任何地方。

**阻塞**的时候线程什么都不能做。

最早的 IO 就是这样的。

**场景**：客户端向服务端发送请求，服务端会为每个客户端建立一个线程来响应，问题来了，如果客户端出现了延时等异常，服务端为客户端建立的线程，就会一直出于等待状态，这个线程就会占用很长时间（因为数据的准备和处理都在这个线程上完成），更糟糕的是，如果有大量并发访问，服务器就会**建立大量线程响应**，引起服务器资源枯竭。

BIO 的网络编程模型基本是 C/S 模型，即两个进程间的通信。

服务端提供 IP 和监听端口，客户端通过连接操作向服务端监听的地址发起连接请求，通过三次握手连接，如果连接成功，双方就可以通过套接字进行通信。

传统的同步阻塞模型开发中，**serverSocket** 负责绑定 ip 地址，启动**监听**端口；**socket** 负责发起**连接操作**，连接成功后，双方通过**输入流和输出流进行同步阻塞式**通信。

采用 BIO 通信模型的服务端，通常由一个独立的 **acceptor 线程**负责监听客户端的连接，它收到客户端连接请求后为**每个客户端创建一个新的线程**进行链路处理，处理完成后，通过输出流返回应答给客户端，**线程销毁**。即典型的**一请求一应答**通信模型。



#### NIO

新的输入/输出 (NIO) 库是在 JDK **1.4** 中引入的，弥补了原来的 I/O 的不足，提供了**高速的、面向块**的 I/O。

##### 1. 流与块

I/O 与 NIO 最重要的区别是数据打包和传输的方式，==I/O 以**流**的方式处理数据，而 NIO 以**块**的方式处理数据==。

**面向流**的 I/O 一次处理**一个字节数据**：一个输入流产生一个字节数据，一个输出流消费一个字节数据。为流式数据创建过滤器非常容易，链接几个过滤器，以便每个过滤器只负责复杂处理机制的一部分。不利的一面是，面向流的 I/O 通常相当慢。

**面向块**的 I/O 一次处理**一个数据块**，按块处理数据比按流处理数据要快得多。但是面向块的 I/O 缺少一些面向流的 I/O 所具有的优雅性和简单性。

I/O 包和 NIO 已经很好地集成了，java.io.\* 已经以 NIO 为基础重新实现了，所以现在它可以利用 NIO 的一些特性。例如，java.io.\* 包中的一些类包含以块的形式读写数据的方法，这使得即使在面向流的系统中，处理速度也会更快。

##### 2. 通道与缓冲区

###### ① 通道 Channel

通道 **Channel** 是对原 I/O 包中的**流的模拟**，可以通过它**读取和写入**数据。

通道与流的不同之处在于，**流**是单向的，只能在**一个方向**上移动(一个流必须是输入流 **InputStream** 或者输出流 **OutputStream** 的子类)，而**通道是==双向==**的，可以用于读、写或者**同时**用于读写。

**通道**包括以下类型，都实现了 **Channel 接口**：

- **FileChannel**：从**文件**中读写数据；
- **DatagramChannel**：通过 **UDP** 读写网络中数据；
- **SocketChannel**：通过 **TCP** 读写网络中数据；
- **ServerSocketChannel**：可以监听新进来的 TCP 连接，对每一个新进来的**连接**都会创建一个 **SocketChannel**。

###### ② 缓冲区 Buffer

发送给一个通道的**所有数据**都必须首先放到**缓冲区**中，同样地，从通道中读取的任何数据都要先**读到缓冲区**中。也就是说，==**不会直接对通道进行读写数据，而是要先经过缓冲区**==。

缓冲区实质上是一个**数组**，但它不仅仅是一个**数组**。缓冲区提供了对数据的**结构化**访问，而且还可以跟踪系统的读/写进程。

缓冲区包括以下常见类型，它们都继承于**抽象类 Buffer** 。

- ByteBuffer、CharBuffer、ShortBuffer、IntBuffer、LongBuffer、FloatBuffer、DoubleBuffer

##### 3. 缓冲区状态变量

有几重要的变量用于描述缓冲区的状态。

- **capacity**：**最大容量**；
- **position**：当前**已经读写**的字节数；
- **limit**：**还可以读写**的字节数。

Buffer 的**操作**一般遵循几个步骤，操作分为**读模式和写模式**：

- 调用 **allocate**() 方法分配缓冲区内存。

- 写入数据到 Buffer。
- 调用 **flip**() 方法。可以将**写模式切换到读模式**。
- 从 Buffer 中**读取**数据。
- 调用 **clear**() 方法，清理 buffer。

状态变量的改变过程举例：

① 通过 **Buffer** 的 **allocate**() 方法进行内存**分配**。新建一个大小为 8 个字节的**缓冲区**，此时 **position** 为 0，而 limit = capacity = 8。**capacity** 变量**不会改变**，下面的讨论会忽略它。

![1563439428762](assets/1563439428762.png)

② **写模式中**，加入首先从输入通道中读取 5 个字节数据**写入**缓冲区中，此时 **position** 为 5，limit 保持**不变**。写模式中的 limit 为可以写入的**最大值**。

![1563439446664](assets/1563439446664.png)

③ 在将**缓冲区**的数据写到**输出通道**之前，需要先调用 **flip()** 方法，这个方法**将 limit 设置为当前 position**，并将 **position 设置为 0**。这可以保证写出的数据**正好**是之前写入缓冲区的数据，因为到了 limit 位置就截止。**flip()** 可以将写模式切换到读模式。**读模式**下，limit 变成写**数据时的 position 值**，代表能够**读出的数据量值**。

![1563439462315](assets/1563439462315.png)

④ 从缓冲区中**取 4 个字节到输出通道**中，此时 position 设为 4。

![1563439481199](assets/1563439481199.png)

⑤ 最后需要调用 **clear()** 方法来**清空缓冲区**，此时 **position 和 limit** 都被设置为**最初位置**。

![1563439494891](assets/1563439494891.png)

##### 4. 选择器 Selector

NIO 常常被叫做**非阻塞 IO**，主要是因为 NIO 在**网络通信**中的非阻塞特性被广泛使用。

NIO 实现了 ==IO **多路复用**中的 **Reactor** 模型==，**一个线程** Thread 使用一个**选择器** Selector 通过**轮询**的方式去**监听多个通道**  Channel 上的事件，从而让一个线程就可以处理多个事件。

通过配置监听的通道 Channel 为**非阻塞**，那么当 Channel 上的 IO 事件还未到达时，就不会进入阻塞状态一直等待，而是**继续轮询**其它 Channel，找到 IO **事件已经到达**的 Channel 并处理事件。

因为创建和切换线程的开销很大，因此使用**一个线程来处理多个事件**而不是一个线程处理一个事件，对于 IO 密集型的应用具有很好地性能。

**一个 Selector** 可以**同时轮询多个** Channel，因为 JDK 使用了 **epoll**() 代替传统的 select() 实现，**没有**最大连接句柄的限制，所以只需要一个线程负责 Selector 轮询，就可以接入**成千上万**的客户端连接。

应该注意的是，只有**套接字 Channel** 才能配置为**非阻塞**，而 FileChannel 不能，为 FileChannel 配置非阻塞也没有意义。

<img src="assets/F-7%20NIO/1582687139737.png" alt="1582687139737" style="zoom:52%;" />

###### ① 创建选择器

```java
Selector selector = Selector.open();
```

###### ② 将通道注册到选择器上

```java
// 打开一个服务端Channel
ServerSocketChannel ssChannel = ServerSocketChannel.open();
// 设置为非阻塞 必须配置
ssChannel.configureBlocking(false);
// 将Channel注册到Selector上
ssChannel.register(selector, SelectionKey.OP_ACCEPT);
```

**通道**必须配置为**非阻塞模式**，否则使用选择器就没有任何意义了，因为如果通道在某个事件上被阻塞，那么服务器就不能响应其它事件，必须等待这个事件处理完毕才能去处理其它事件，显然这和选择器的作用背道而驰。

在将通道**注册**到选择器上时，还需要指定要**注册的具体事件**，主要有以下几类：

- SelectionKey.**OP_CONNECT**：连接就绪
- SelectionKey.**OP_ACCEPT**：接收就绪
- SelectionKey.**OP_READ**：读就绪
- SelectionKey.**OP_WRITE**：写就绪

它们在 SelectionKey 的定义如下：

```java
public static final int OP_READ = 1 << 0;
public static final int OP_WRITE = 1 << 2;
public static final int OP_CONNECT = 1 << 3;
public static final int OP_ACCEPT = 1 << 4;
```

可以看出每个事件可以被当成一个**位域**，从而组成**事件集整数**。例如：

```java
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```

###### ③ 监听事件

```java
int num = selector.select();
```

使用 select() 来**监听**到达的事件，它会**一直阻塞**直到有**至少一个**事件到达，可能有**多个**事件到达。

###### ④ 获取到达的事件

Selector 在有通道**事件到达**的时候返回 **SelectionKey 的集合**，之后可以进行数据处理。这里循环**遍历**已经选择键集合中的**每一个键**（因为可能有多个通道的事件需要处理），并检测各个**键所对应的通道事件的类型**进而做出不同的处理。

```java
// 遍历SelectionKey事件
Set<SelectionKey> keys = selector.selectedKeys();
Iterator<SelectionKey> keyIterator = keys.iterator();
while (keyIterator.hasNext()) {
    SelectionKey key = keyIterator.next();
    // 判断事件类型
    if (key.isAcceptable()) {
        // ...
    } else if (key.isReadable()) {
        // ...
    }
    keyIterator.remove();
}
```

###### ⑤ 事件循环

因为一次 select() 调用**不能处理完所有的事件**，并且服务器端有可能需要一直监听事件，因此服务器端**处理事件**的代码一般会放在一个**死循环**内。

```java
while (true) {
    int num = selector.select();
    Set<SelectionKey> keys = selector.selectedKeys();
    Iterator<SelectionKey> keyIterator = keys.iterator();
    while (keyIterator.hasNext()) {
        SelectionKey key = keyIterator.next();
        if (key.isAcceptable()) {
            // ...
        } else if (key.isReadable()) {
            // ...
        }
        keyIterator.remove();
    }
}
```

##### 5. 文件 NIO

使用 NIO 读取数据基本流程：

- 从 FileInputStream 获取 Channel。
- 创建 Buffer。
- 将数据从 Channel 读取到 Buffer 中。

以下展示了使用 **NIO 快速复制文件**的实例：

```java
public static void fastCopy(String src, String dist) throws IOException {

    /* 获得源文件的输入字节流 */
    FileInputStream fin = new FileInputStream(src);

    /* 获取输入字节流的文件通道 */
    FileChannel fcin = fin.getChannel();

    /* 获取目标文件的输出字节流 */
    FileOutputStream fout = new FileOutputStream(dist);

    /* 获取输出字节流的文件通道 */
    FileChannel fcout = fout.getChannel();

    /* 为缓冲区分配 1024 个字节 */
    ByteBuffer buffer = ByteBuffer.allocateDirect(1024);

    while (true) {

        /* 从输入通道中读取数据到缓冲区中 */
        int r = fcin.read(buffer);

        /* read() 返回 -1 表示 EOF */
        if (r == -1) {
            break;
        }

        /* 切换读写 */
        buffer.flip();

        /* 把缓冲区的内容写入输出文件中 */
        fcout.write(buffer);

        /* 清空缓冲区 */
        buffer.clear();
    }
}
```

##### 6. 套接字 NIO 实例

NIO服务器

```java
public class NIOServer {

    public static void main(String[] args) throws IOException {
		// 创建选择器
        Selector selector = Selector.open();	
		// 创建Channel
        ServerSocketChannel ssChannel = ServerSocketChannel.open();
        // 设置Channel为非阻塞 必须
        ssChannel.configureBlocking(false);
        // 注册Channel到Selector上并制定事件
        ssChannel.register(selector, SelectionKey.OP_ACCEPT);
		// 通过Channel获取服务Socket
        ServerSocket serverSocket = ssChannel.socket();
        InetSocketAddress address = new InetSocketAddress("127.0.0.1", 8888);
        // 绑定端口
        serverSocket.bind(address);
		// 死循环监听
        while (true) {
			// 调用完成则阻塞 直到事件到达
            selector.select();
            // 遍历SelectionKey
            Set<SelectionKey> keys = selector.selectedKeys();
            Iterator<SelectionKey> keyIterator = keys.iterator();
			// 循环遍历就绪的事件集
            while (keyIterator.hasNext()) {
				
                SelectionKey key = keyIterator.next();
				// 判断SelectionKey的类型
                if (key.isAcceptable()) {
                    // 得到事件的channel
                    ServerSocketChannel ssChannel1 = (ServerSocketChannel) key.channel();
                    // 服务器会为每个新连接创建一个 SocketChannel
                    SocketChannel sChannel = ssChannel1.accept();
                    sChannel.configureBlocking(false);

                    // 这个新连接主要用于从客户端读取数据
                    sChannel.register(selector, SelectionKey.OP_READ);

                } else if (key.isReadable()) {
					
                    SocketChannel sChannel = (SocketChannel) key.channel();
                    System.out.println(readDataFromSocketChannel(sChannel));
                    sChannel.close();
                }

                keyIterator.remove();
            }
        }
    }

    // 从socket通道中读取数据：读取的过程可以结合缓存区状态变量的图示过程
    private static String readDataFromSocketChannel(SocketChannel sChannel) throws IOException {
		// 分配缓冲区
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        StringBuilder data = new StringBuilder();
        while (true) {
            buffer.clear();
            // 从channel读数据到buffer
            int n = sChannel.read(buffer);
            if (n == -1) {
                break;
            }
            // flip方法改变为读模式
            buffer.flip();
            // 此时limit就是buffer中实际有的数据数量
            int limit = buffer.limit();
            char[] dst = new char[limit];
            // 依次读取buffer的数据
            for (int i = 0; i < limit; i++) {
                dst[i] = (char) buffer.get(i);
            }
            data.append(dst);
            // 清理缓冲区
            buffer.clear();
        }
        return data.toString();
    }
}
```

NIO 客户端：NIO 这边还是一般的连接方式。

```java
public class NIOClient {

    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1", 8888);
        OutputStream out = socket.getOutputStream();
        String s = "hello world";
        out.write(s.getBytes());
        out.close();
    }
}
```

##### 7. 对比

**NIO** 与普通 BIO 的区别主要有以下两点：

- NIO 是**非阻塞**的；
- NIO 面向**块**，I/O 面向流。
- NIO 对于处理有**大量连接**的场景比较好，比如 P2P 网络，聊天服务器等。而 BIO 对于少量连接并发送大量数据的场景是比较适合的。



#### AIO

##### 1. 概述

AIO（Asynchronous IO）是 Java7 才实现的真正**异步**的 IO。它把 IO **读写操作完全交给了操作系统**。

AIO 最大的一个特性就是**异步能力**，这种能力对 socket 与文件 I/O 都起作用。AIO 其实是一种在**读写操作结束之前允许进行其他操作的 I/O 处理**。AIO 是对 JDK1.4 中提出的**同步非阻塞 I/O(NIO)** 的进一步增强。

Jdk7 主要增加了三个新的**异步通道**:

- **AsynchronousFileChannel**: 用于**文件**异步读写；
- **AsynchronousSocketChannel**: **客户端**异步 socket；
- **AsynchronousServerSocketChannel**: **服务器**异步 socket。

因为 AIO 的实施需**充分调用 OS** 参与，IO 需要操作系统支持、并发也同样需要操作系统的支持，所以性能方面不同操作系统差异会比较明显。

##### 2. 实例

下面以一个最简单的 Time 服务的例子演示如何使用异步 I/O。 客户端连接到服务器后服务器就发送一个当前的时间字符串给客户端。 客户端毋须发送请求。 逻辑很简单。

**Server**

```java
public class Server {
    private static Charset charset = Charset.forName("US-ASCII");
    private static CharsetEncoder encoder = charset.newEncoder();

    public static void main(String[] args) throws Exception {
        AsynchronousChannelGroup group = AsynchronousChannelGroup.withThreadPool(Executors.newFixedThreadPool(4));
        AsynchronousServerSocketChannel server = AsynchronousServerSocketChannel.open(group).bind(new InetSocketAddress("0.0.0.0", 8013));
        server.accept(null, new CompletionHandler<AsynchronousSocketChannel, Void>() {
            @Override
            public void completed(AsynchronousSocketChannel result, Void attachment) {
                server.accept(null, this); // 接受下一个连接
                try {
                    String now = new Date().toString();
                    ByteBuffer buffer = encoder.encode(CharBuffer.wrap(now + "\r\n"));
                    //result.write(buffer, null, new CompletionHandler<Integer,Void>(){...}); //callback or
                    Future<Integer> f = result.write(buffer);
                    f.get();
                    System.out.println("sent to client: " + now);
                    result.close();
                } catch (IOException | InterruptedException | ExecutionException e) {
                    e.printStackTrace();
                }
            }
            @Override
            public void failed(Throwable exc, Void attachment) {
                exc.printStackTrace();
            }
        });
        group.awaitTermination(Long.MAX_VALUE, TimeUnit.SECONDS);
    }
}
```

这个例子使用了两种方式。 `accept`使用了**回调**的方式， 而发送数据使用了`future`的方式。

**Client**

```java
public class Client {
    public static void main(String[] args) throws Exception {
        AsynchronousSocketChannel client = AsynchronousSocketChannel.open();
        Future<Void> future = client.connect(new InetSocketAddress("127.0.0.1", 8013));
        future.get();

        ByteBuffer buffer = ByteBuffer.allocate(100);
        client.read(buffer, null, new CompletionHandler<Integer, Void>() {
            @Override
            public void completed(Integer result, Void attachment) {
                System.out.println("client received: " + new String(buffer.array()));

            }
            @Override
            public void failed(Throwable exc, Void attachment) {
                exc.printStackTrace();
                try {
                    client.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        });

        Thread.sleep(10000);
    }
}
```

客户端也使用了两种方式， `connect`使用了 **future** 方式，而接收数据使用了**回调**的方式。

##### 3. Netty AIO

Netty 也支持 **AIO** 并提供了相应的类： `AioEventLoopGroup`,`AioCompletionHandler`, `AioServerSocketChannel`, `AioSocketChannel`， `AioSocketChannelConfig`。
其它使用方法和 NIO 类似。



#### 对比

- **BIO：同步并阻塞**，服务器实现模式为**一个连接一个线程**，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。

- **NIO：同步非阻塞**，服务器实现模式为**一个线程多个连接**，即客户端发送的**连接请求**都会注册到**多路复用器**上，多路复用器轮询到连接有 I/O 请求时才启动一个线程进行**处理**。

- **AIO：异步非阻塞**，服务器实现模式为**一个有效请求一个线程**，客户端的 I/O 请求都是**由 OS 先完成了再通知服务器**应用去启动线程进行处理。



#### BIO/NIO/AIO适用场景分析

- BIO 方式适用于**连接数目比较小且固定**的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4 以前的唯一选择，但程序直观简单易理解。

- NIO 方式适用于**连接数目多且连接比较短**（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，J DK1.4 开始支持。

- AIO 方式使用于**连接数目多且连接比较长**（重操作）的架构，比如相册服务器，充分调用 **OS 参与**并发操作，编程比较复杂，JDK7 开始支持。



#### **Reactor模式和Proactor模式**

**其实阻塞与非阻塞都可以理解为同步范畴下才有的概念，对于异步，就不会再去分阻塞非阻塞。对于用户进程，接到异步通知后，就直接操作进程用户态空间里的数据好了。**

- 两种 **IO 多路复用**方案: **Reactor and Proactor。**
- Reactor 模式是基于**同步I/O**的，而 Proactor 模式是和**异步** I/O 相关的。
- Reactor：能收了你跟俺说一声。Proactor: 你给我收十个字节，收好了跟俺说一声。

##### 1. Reactor模式

首先来看看 Reactor 模式，Reactor 模式应用于**同步 I/O** 的场景。我们分别以读操作和写操作为例来看看 Reactor 中的具体步骤： 
读取操作： 

- 应用程序注册读就绪事件和相关联的事件处理器 
- 事件分离器等待事件的发生 
- 当发生读就绪事件的时候，事件分离器调用第一步注册的事件处理器 

- 事件处理器首先执行实际的读取操作，然后根据读取到的内容进行进一步的处理 

写入操作类似于读取操作，只不过第一步注册的是写就绪事件。 

##### 2. Proactor模式

下面我们来看看 Proactor 模式中读取操作和写入操作的过程： 

读取操作： 

- 应用程序初始化一个异步读取操作，然后注册相应的事件处理器，此时事件处理器不关注读取就绪事件，而是关注读取完成事件，这是区别于Reactor的关键。 

- 事件分离器等待读取操作完成事件 

- 在事件分离器等待读取操作完成的时候，操作系统调用内核线程完成读取操作（异步IO都是操作系统负责将数据读写到应用传递进来的缓冲区供应用程序操作，操作系统扮演了重要角色），并将读取的内容放入用户传递过来的缓存区中。这也是区别于Reactor的一点，Proactor中，应用程序需要传递缓存区。 

- 事件分离器捕获到读取完成事件后，激活应用程序注册的事件处理器，事件处理器直接从缓存区读取数据，而不需要进行实际的读取操作。 

Proactor 中写入操作和读取操作类似，只不过感兴趣的事件是**写入完成**事件。 

从上面可以看出，Reactor 和 Proactor 模式的主要区别就是**真正的读取和写入操作是有谁来完成**的，Reactor 中需要应用程序自己读取或者写入数据，而 Proactor 模式中，应用程序不需要进行实际的读写过程，它只需要从缓存区读取或者写入即可，操作系统会读取缓存区或者写入缓存区到真正的 IO 设备.。

综上所述，同步和异步是相对于应用和内核的交互方式而言的，同步需要主动去询问，而异步的时候内核在 IO 事件发生的时候通知应用程序，而阻塞和非阻塞仅仅是系统在调用系统调用的时候函数的实现方式而已。

