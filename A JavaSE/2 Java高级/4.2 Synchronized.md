[TOC]

### Synchronized

Java 提供了两种锁机制来控制多个线程对共享资源的**互斥访问**，第一个是 JVM 实现的 **synchronized**，而另一个是 JDK 实现的 **ReentrantLock**(见后面)。

使用了**synchronized** 就变成了原子操作。

#### synchronized 用法

##### 1. 同步代码块与实例方法

synchronized 保护的是**对象**。

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

它只作用于==同一个对象==，如果调用两个对象上的同步代码块，就不会进行同步。即保护的是同一个对象的方法调用。再具体来说，synchronized 实例方法保护的是当前实例对象，**即 this**。this 对象有一个**锁**和一个**等待队列**，锁只能被一个线程持有，其他线程需要锁时会尝试获取，没获取到就进入等待队列等待，并进入 BLOCKED 状态。

对于以下代码，使用 ExecutorService 执行了两个线程，由于调用的是同一个对象的同步代码块，因此这两个线程会进行同步，当一个线程进入同步语句块时，另一个线程就必须**等待**。

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

##### 2. 同步一个静态方法

synchronized 保护的是**对象**。对于实例方法，保护的是当前实例对象 this，而对于静态方法，保护的是**类对象**。即 StaticCounter.class。每个对象都有一个锁与等待队列，类对象也不例外。

synchronized 静态方法和 synchronized 实例方法保护的是不同的对象，不同的两个线程可以一个执行 synchronized 静态方法，另一个执行 synchronized  实例方法。

```java
public class StaticCounter {
    private static int count = 0;
    public static synchronized void incr() {
        count ++;
    }
}
```

##### 3. 同步一个类

```java
public void func() {
    synchronized (SynchronizedExample.class) {
        // ...
    }
}
```

作用于**整个类**，也就是说两个线程调用同一个类的**不同对象**上的这种同步语句，也会进行同步。

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



#### synchronized 的特性

##### 可重入性

synchronized 是**可重入**的，即同一个执行线程，它获得了锁之后，可以直接调用其他需要同样锁的代码，无需等待。如在一个 synchronized 实例方法内可以直接调用其他 synchronized 实例方法。

可重入是通过记录锁的持有线程和持有**数量**来实现的。

##### 内存可见性

synchronized 可以实现原子操作，避免出现竞态条件。同时还可以解决内存可见性问题，在释放锁时，所有的写入都会写回内存，获得锁后，都会从内存读取最新数据。

如果只是为了保证内存可见性，使用 synchronized 成本有点高，可以使用轻量级的 volatile 关键字，但是不能解决竞态条件的问题。

```java
private volatile boolean switch = false;
```

##### 死锁

应该避免在持有一个锁的同时去申请另一个锁，如果确实需要多个锁，所有代码都应该按照相同的顺序去申请锁。

可以使用显示锁 Lock 的方式来解决部分死锁问题，它支持尝试获取锁和带时间限制的获取锁方法，使用这些方法可以在获取不到锁的时候放弃已经持有的锁。

##### 同步容器

类 Collections 中有一些方法可以返回线程安全的**同步容器**（不是并发容器）。它是给所有容器方法都加上 **synchronized** 关键字实现的。

但是给所有的方法加 synchronized 关键词时所有的方法必须使用**相同的锁**（），不然可能造成伪同步的问题。

同步容器的单个操作是安全的，但是**迭代操作不是**。如果在遍历容器时发生了结构性变化，就会抛出异常。同步容器没有解决这个问题，要避免这个异常，需要在遍历时给整个容器对象加锁。

同步容器不好，可以使用并发容器。并发容器都是线程安全的，且没有使用 synchronized 关键字，且没有迭代问题，直接支持一些复合操作。



