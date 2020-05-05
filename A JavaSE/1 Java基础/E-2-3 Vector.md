[TOC]

### Vector

#### 同步

它的实现与 **ArrayList** 类似，但是使用了 **synchronized** 进行同步。在**方法**上加 **synchronized** 。因此是==**线程安全**==的。

```java
public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}

public synchronized E get(int index) {
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);
    return elementData(index);
}
```

#### 与ArrayList的比较

- **ArrayList** 线程**不安全**，而 Vector 类采用了**同步机制**保证了线程**安全**。

- Vector 是**同步**的，因此==**开销**==就比 ArrayList 要**大**，访问速度更慢。最好使用 ArrayList 而不是 Vector，因为同步操作完全可以由程序员自己来控制；
- Vector 每次扩容请求其大小的 **2 倍**空间，而 ArrayList 是 **1.5 倍**。
- 内部也有 **modCount** 记录**结构性变化次数**。

#### 替代方案

可以使用 `Collections.synchronizedList();` 得到一个**线程安全的 ArrayList**。但是也是使用了 synchronized  锁机制。

```java
List<String> list = new ArrayList<>();
List<String> synList = Collections.synchronizedList(list);	// ※
```

也可以使用 JUC 并发包下的 **CopyOnWriteArrayList** 类。

```java
List<String> list = new CopyOnWriteArrayList<>();
```













