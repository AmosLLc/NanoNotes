[TOC]

### Synchronized 锁

#### 0 概述

Java 提供了**两种锁机制**来控制多个线程对共享资源的**互斥访问**，第一个是 JVM 实现的 **synchronized**，而另一个是 JDK 实现的 **ReentrantLock**(见后面)。

使用了**synchronized** 就变成了原子操作。



#### 1 synchronized 基本用法

##### ① 同步代码块与实例方法

synchronized 保护的是==**对象**==。

```java
// 同步代码块
public void func() {
    synchronized (this) {
        // ...
    }
}

// 同步实例方法
public synchronized void func () {
    // ...
}
```

它只作用于==**同一个对象**==，如果调用两个对象上的同步代码块，就**不会**进行同步，想一下如果是多个对象各自独立操作，其实不会存在竞争。即保护的是**同一个对象的方法调用**。再具体来说，synchronized 实例方法保护的是当前实例对象，**即 ==this==对象**。this 对象有一个**锁**和一个**等待队列**，锁只能被一个线程持有，其他线程需要锁时会尝试获取，没获取到就进入**等待队列**等待，并进入 **BLOCKED** 状态。

对于以下代码，使用 ExecutorService 执行了两个线程，由于调用的是**同一个对象的同步代码块**，因此这两个线程会进行同步，当一个线程进入**同步语句块**时，另一个线程就必须**等待**。

```java
public class SynchronizedExample {

    public void func1() {
        synchronized (this) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
```

```java
public static void main(String[] args) {
    // 作用于同一个e1对象 需要等待
    SynchronizedExample e1 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func1());
    executorService.execute(() -> e1.func1());
}
```

```html
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

对于以下代码，两个线程调用了**不同对象**的同步代码块，两个对象**各自拥有**自己的锁和等待队列，因此这两个线程就**不需要同步**。从输出结果可以看出，两个线程交叉执行。

```java
public static void main(String[] args) {
    // 由于同步的是代码块 所以使用两个不同对象的锁不用同步
    SynchronizedExample e1 = new SynchronizedExample();
    SynchronizedExample e2 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func1());
    executorService.execute(() -> e2.func1());
}
```

```html
0 0 1 1 2 2 3 3 4 4 5 5 6 6 7 7 8 8 9 9
```

##### ② 同步一个静态方法

synchronized 保护的是**对象**。对于**实例方法**，保护的是当前实例对象 this，而对于静态方法，保护的是==**类对象**==。即 **StaticCounter.class**。每个对象都有一个锁与等待队列，类对象也不例外。

synchronized 静态方法和 synchronized 实例方法保护的是不同的对象，**不同的两个线程可以一个执行 synchronized 静态方法，另一个执行 synchronized  实例方法。**因为类对象和实例对象属于不同的对象，两者都有其各自的锁与等待队列。

```java
public class StaticCounter {
    private static int count = 0;
    // 静态方法加锁
    public static synchronized void incr() {
        count ++;
    }
}
```

锁住静态方法保证同时只能有一个线程在使用这个静态方法资源，如果有竞争就会等待。

##### ③ 同步一个类

```java
public void func() {
    synchronized (SynchronizedExample.class) {
        // ...
    }
}
```

作用于==**整个类**==，也就是说两个线程调用**同一个类**的**不同对象**上的这种同步语句，也会进行同步。

```java
public class SynchronizedExample {
    public void func2() {
        synchronized (SynchronizedExample.class) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }
}
```

```java
public static void main(String[] args) {
    // 由于是锁住整个类 所以两个不同对象也需要等待
    SynchronizedExample e1 = new SynchronizedExample();
    SynchronizedExample e2 = new SynchronizedExample();
    ExecutorService executorService = Executors.newCachedThreadPool();
    executorService.execute(() -> e1.func2());
    executorService.execute(() -> e2.func2());
}
```

```html
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```





#### 2 synchronized 的特性

##### ① 可重入性

synchronized 是**可重入**的，即**同一个执行线程**，它获得了对象锁之后，可以**直接调用其他需要同样锁的代码**，无需等待。如在一个 synchronized **实例方法**内可以**直接调用**其他 synchronized **实例方法**。

可重入是通过记录锁的持有线程和持有**数量**来实现的。

##### ② 内存可见性

synchronized 可以实现**原子操作**，避免出现**竞态条件**。同时还可以**解决内存可见性**问题，在释放锁时，所有的写入都会写回内存，获得锁后，都会从内存读取最新数据。

如果只是为了保证内存可见性，使用 synchronized 成本有点高，可以使用轻量级的 volatile 关键字，但是不能解决竞态条件的问题。

```java
private volatile boolean switch = false;
```

##### ③ 死锁

应该避免在持有一个锁的同时去申请另一个锁，如果确实需要多个锁，所有代码都应该**按照相同的顺序**去申请锁。

可以使用**显示锁 Lock** 的方式来解决部分死锁问题，它支持尝试获取锁和带时间限制的获取锁方法，使用这些方法可以在获取不到锁的时候放弃已经持有的锁。



#### 3 解决静态条件问题

可以使用 **synchronized** 互斥锁来保证操作的**原子性**，以解决前述的静态条件问题。它对应的内存间交互操作为：lock 和 unlock，在虚拟机实现上对应的字节码指令为 **monitorenter 和 monitorexit**。

```java
public class AtomicSynchronizedExample {
    private int cnt = 0;
	// 对自增方法加锁 保证只有一个线程能操作此方法
    public synchronized void add() {
        cnt++;
    }

    public synchronized int get() {
        return cnt;
    }
}
```

```java
public static void main(String[] args) throws InterruptedException {
    final int threadSize = 1000;
    AtomicSynchronizedExample example = new AtomicSynchronizedExample();
    final CountDownLatch countDownLatch = new CountDownLatch(threadSize);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < threadSize; i++) {
        executorService.execute(() -> {
            example.add();
            countDownLatch.countDown();
        });
    }
    countDownLatch.await();
    executorService.shutdown();
    System.out.println(example.get());
}
```

输出为：

```html
1000
```



