[TOC]

### HashSet

#### 概述

- HashSet 的底层通过 **HashMap** 实现的。而 HashMap 在1.7之前使用的是**数组 + 链表**实现，在1.8 + 使用的**数组 + 链表 + 红黑树**实现。其实也可以这样理解，HashSet 的底层实现和 HashMap 使用的是相同的方式，因为 Map 是无序的，因此 HashSet 也**无法保证顺序**。
- Set **没有重复元素**。相同的元素添加进去也只会保留一份。
- HashSet 的方法，也是借助 HashMap 的方法来实现的。
- 可以高效的添加、删除元素、判断元素是否存在，效率都为O(1)。
- transient 修饰存储数据的数组可以**保证其序列化时不被序列化**。



#### 源码解析

HashSet 内部使用 **HashMap** 存储数据，所以很多操作都是围绕 HashMap 开展的。

```java
public class HashSet<E> extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {

    static final long serialVersionUID = -5024744406713321676L; // 序列化版本号

    // HashMap变量，用于存放HashSet的值 
    private transient HashMap<E,Object> map;  

    private static final Object PRESENT = new Object(); // map中的值

    // 构造方法
    public HashSet() {
        map = new HashMap<>();
    }

    // 构造方法，将指定的集合转化为HashSet
    public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }

    // 构造方法，指定初始化的大小和负载因子
    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }

    // 指定初始化大小
    public HashSet(int initialCapacity) {
        map = new HashMap<>(initialCapacity);
    }

    // 构造方法，采用default修饰，只能是同一个包下的成员访问。包不相同无法访问
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }

    // HashSet的遍历操作
    // 通过这个方法可以发现，HashSet调用了HashMap存放，因为HashSet并不是键值对存储，所以它只是把它的值做了Map中的键，在遍历HashSet的集合元素时，实际上是遍历的Map中Key的集合。
    public Iterator<E> iterator() {
        return map.keySet().iterator();
    }

    // 返回集合中元素的容量
    public int size() {
        return map.size();
    }

    // 判断是否为空
    public boolean isEmpty() {
        return map.isEmpty();
    }

    // 是否包含指定的元素
    public boolean contains(Object o) {
        return map.containsKey(o);
    }

    // 添加元素，添加的元素作为了Map中的key,value使用了一个常量表示
    public boolean add(E e) {
        return map.put(e, PRESENT) == null;
    }

    // 删除元素
    public boolean remove(Object o) {
        return map.remove(o) == PRESENT;
    }

    // 清空集合
    public void clear() {
        map.clear();
    }

    // 克隆方法
    public Object clone() {
        try {
            HashSet<E> newSet = (HashSet<E>) super.clone();
            newSet.map = (HashMap<E, Object>) map.clone();
            return newSet;
        } catch (CloneNotSupportedException e) {
            throw new InternalError();
        }
    }

    // 写入输出流操作 序列化 与ArrayList类似
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        // Write out any hidden serialization magic
        s.defaultWriteObject();

        // Write out HashMap capacity and load factor
        s.writeInt(map.capacity());
        s.writeFloat(map.loadFactor());

        // Write out size
        s.writeInt(map.size());

        // Write out all elements in the proper order.
        for (E e : map.keySet())
            s.writeObject(e);
    }

    // 从输入流中读取对象 反序列化
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        // Read in any hidden serialization magic
        s.defaultReadObject();

        // Read in HashMap capacity and load factor and create backing HashMap
        int capacity = s.readInt();
        float loadFactor = s.readFloat();
        map = (((HashSet)this) instanceof LinkedHashSet ?
               new LinkedHashMap<E, Object>(capacity, loadFactor) :
               new HashMap<E, Object>(capacity, loadFactor));

        // Read in size
        int size = s.readInt();

        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            E e = (E) s.readObject();
            map.put(e, PRESENT);
        }
    }
}
```

HashSet 使用成员对象来计算 hashCode 值，对于两个对象来说 hashCode **可能相同**，所以 equals() 方法用来**判断对象**的**相等性**，如果两个对象不同的话，那么返回 false。



> **HashSet** 如何**检查重复**？

当你把对象加入 HashSet 时，HashSet 会先计算对象的 **hashcode 值**来判断对象加入的位置，同时也会与该位置其他已经加入的对象的 hashcode 值作比较，如果没有相符的 hashcode，HashSet 会假设对象没有重复出现。但是如果发现有相同 hashcode 值的对象，这时会调用 **`equals()`方法**来检查 hashcode 相等的对象是否真的相同。如果两者相同，HashSet 就不会让其加入操作成功。如果不同的话，就会重新散列到其他位置。（摘自我的 Java 启蒙书《Head first java》第二版）。这样我们就大大减少了 equals 的次数，相应就大大提高了执行速度。

总计：先 **hashCode**，再 **equals**。



#### 比较

##### 1. HashMap与HashSet比较

HashSet 底层就是基于 HashMap 实现的。（HashSet 的源码非常非常少，因为除了 `clone() `、`writeObject()`、`readObject()`是 HashSet 自己不得不实现之外，其他方法都是直接调用 HashMap 中的方法。

|              HashMap              |                           HashSet                            |
| :-------------------------------: | :----------------------------------------------------------: |
|          实现了 Map 接口          |                        实现 Set 接口                         |
|            存储键值对             |                          仅存储对象                          |
|   调用 `put()`向 map 中添加元素   |              调用 `add()`方法向 Set 中添加元素               |
| HashMap 使用键（Key）计算Hashcode | HashSet 使用成员对象来计算 hashcode 值，对于两个对象来说 hashcode 可能相同，所以 equals() 方法用来判断对象的相等性 |