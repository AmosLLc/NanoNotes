[TOC]

### WeakHashMap

#### 原理解析

WeakHashMap 的 Entry **继承自 WeakReference**，被 WeakReference 关联的对象在**下一次垃圾**回收时会**被回收**。（弱引用）。

WeakHashMap 主要用来**实现缓存**，通过使用 WeakHashMap 来引用**缓存对象**，由 JVM 对这部分**缓存进行回收**。

```java
private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V>
```

WeakHashMap 的**键是“弱键”**。在 WeakHashMap 中，当某个键不再正常使用时，会被从 WeakHashMap 中被**自动移除**。更精确地说，对于一个给定的键，其映射的存在**并不阻止垃圾回收器**对该键的丢弃，这就使该键成为可终止的，被终止，然后被回收。某个键被终止时，它对应的键值对也就从映射中有效地移除了。

弱键大致上就是**通过 WeakReference 和 ReferenceQueue 实现的**。 WeakHashMap 的 key 是“弱键”，即是 WeakReference 类型的；ReferenceQueue 是一个**队列**，它会保存被 GC 回收的“弱键”。步骤如下：

- 新建 WeakHashMap，将“**键值对**”添加到 WeakHashMap 中。实际上，WeakHashMap 是通过**数组 table **保存 Entry (键值对)；每一个 Entry 实际上是一个单向链表，即 Entry 是键值对链表，类似 HashMap。
- 当**某“弱键”不再被其它对象引用**，并**被 GC 回收**时。在 GC 回收该“弱键”时，**这个“弱键”也同时会被添加到 ReferenceQueue(queue) 队列**中。
- 当**下一次**我们需要操作 WeakHashMap 时，会先**同步 table** 和 **queue**。table 中保存了**全部**的键值对，而 queue 中保存被 GC 回收的键值对；同步它们，就是**删除 table 中被 GC 回收的键值对**。



#### ConcurrentCache

Tomcat 中的 **ConcurrentCache** 使用了 WeakHashMap 来实现**缓存**功能。

**ConcurrentCache** 采取的是**分代缓存**：

- **经常使用**的对象放入 **eden** 中，eden 使用 **ConcurrentHashMap** 实现，不用担心会被回收（伊甸园）；
- **不常用**的对象放入 **longterm**，longterm 使用 **WeakHashMap** 实现，这些**老对象**会被垃圾收集器回收。
- 当调用  **get**() 方法时，会先从 **eden** 区获取，如果没有找到的话再到 **longterm** 获取，当从 longterm **获取到**就把对象**放入 eden** 中，从而保证经常被访问的节点**不容易被回收**。相当于更新缓存。
- 当调用 **put**() 方法时，如果 eden 的大小超过了 size，那么就将 eden 中的所有对象**都放入** longterm 中，利用虚拟机回收掉一部分不经常使用的对象。

```java
public final class ConcurrentCache<K, V> {

    private final int size;
    // 伊甸区
    private final Map<K, V> eden;

    private final Map<K, V> longterm;

    public ConcurrentCache(int size) {
        this.size = size;
        this.eden = new ConcurrentHashMap<>(size);
        this.longterm = new WeakHashMap<>(size);
    }

    public V get(K k) {
        V v = this.eden.get(k);
        if (v == null) {
            v = this.longterm.get(k);
            if (v != null)
                this.eden.put(k, v);
        }
        return v;
    }

    public void put(K k, V v) {
        if (this.eden.size() >= size) {
            this.longterm.putAll(this.eden);
            this.eden.clear();
        }
        this.eden.put(k, v);
    }
}
```

















