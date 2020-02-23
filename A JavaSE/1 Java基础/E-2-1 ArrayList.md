[TOC]



### ArrayList

#### 1 概述

- ArrayList 是一个采用类型参数的**泛型数组列表类**。保存元素的**类型**放在尖括号中。使用时需要实例化泛型参数。
- 如果添加元素时数组满了，会自动创建更大的数组并把所有对象从小数组**拷贝**到大数组，自动扩容。
- ArrayList 实现了Serializable, Cloneable, Iterable\<E>, Collection\<E>, List\<E>, RandomAccess 等接口。
- ArrayList 对象**不能存储基本类型**，只能存储**引用类型**的数据。类似 \<int> 不能写，想要存储基本类型数据，<> 中的数据类型，需要使用==基本类型包装类==，如\<Integer> 也可能用到基本数据类型，JVM会根据场景进行自动拆箱、自动装箱。
- 插入与删除元素效率低，因为需要移动其它元素。
- ArrayLis t中的操作**不是线程安全**的。所以，建议在单线程中才使用 ArrayList，而在多线程中可以选择 Vector或者 **CopyOnWriteArrayList**。
- ArrayList 实现 java.io.Serializable 的方式。当写入到输出流时，先写入“容量”，再依次写入“每一个元素”；当读出输入流时，先读取“容量”，再依次读取“每一个元素”。
- 可以**随机访问**，按照索引进行访问的效率很高，效率为 O(1)。
- 除非数组已经排序，否则按照内容查找元素效率较低，效率为 O(N)。
- 添加元素效率视情况而定，可能会面临数组**扩容**与内容**复制**等开销问题。
- 插入与删除元素效率较低，因为需要移动元素。
- ArrayList 序列化时不会全部序列化其存储数组（因为不一定全部存完了的），而是自己实现了序列化与反序列化方法。

```java
ArrayList<Employee> staff = new ArrayList<Employee>();
// 右边的类型参数可省并指定初始大小 一定要写，避免多次自动扩容影响性能
ArrayList<Employee> staff = new ArrayList<>(100);  
// 遍历方法
for(Employee e : staff){
    e.raiseMoney(300);
}

for(int i = 0; i < staff.size(); i++){
    staff.get(i).raiseMoney(300);
}
```



#### 2 ArrayList类 API

```java
public boolean add(E obj); 				// 将指定元素添加到此集合的尾部
public boolean add(int index, E obj); 	// 在指定位置插入元素，后面的元素往后移动
public E remove(int index); 			// 删除指定位置上的元素，返回被删除的元素
public E get(int index);    			// 返回指定位置上的元素
public int size();  					// 返回此集合中的元素数目
public void trimToSize();  	// 将数组列表的存储容量削减到当前尺寸，确保数组不会有新元素添加的时候调用
public void set(int index, E obj);		// 设置数组列表指定位置的值，覆盖原有内容。
```



#### 3 源码分析

下面的分析基于 JDK8。最新的 ArrayList 源码是有**改动**的。

因为 ArrayList 是基于**数组**实现的，所以支持**快速随机访问**。RandomAccess 接口标识着该类**支持快速随机访问**，RandomAccess 是一个**标记接口**，没有任何方法。

##### ① 基本属性

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    // 版本号
    private static final long serialVersionUID = 8683452581122892189L;
    // 默认容量
    private static final int DEFAULT_CAPACITY = 10;
    // 空对象数组
    private static final Object[] EMPTY_ELEMENTDATA = {};
    // 缺省空对象数组
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    // 存放元素数组
    transient Object[] elementData;
    // 实际元素大小，默认为0
    private int size;
    // 最大数组容量
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
}
```

存放元素数组为 elementData，其默认大小为 DEFAULT_CAPACITY = 10。

![1582447809913](assets/E-2-1%20ArrayList/1582447809913.png)

##### ② 添加元素

添加元素时使用 **ensureCapacityInternal() 方法**来保证容量足够，如果不够时，需要使用 **grow() 方法**进行扩容，新容量的大小为 `oldCapacity + (oldCapacity >> 1)`，也就是旧容量的 **1.5 倍**。elementData 数组会随着实际元素个数的增多而重新分配。

扩容操作需要调用 `Arrays.copyOf()` 把原数组整个复制到新数组中，这个操作代价很高，因此最好在创建 ArrayList 对象时就**指定**大概的容量大小，**减少扩容操作的次数**。

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);   // Increments modCount!!
    elementData[size++] = e;			// size 用于记录实际的元素个数	
    return true;
}

// 确保数组容量足够
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    // 表示内部修改次数
    modCount++;
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

// 增加数组容量
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    // 右移一位相当于除2，因此扩容为原来的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

##### ③ 删除元素

需要调用 **System.arraycopy()** 将 index + 1 后面的元素都复制到 index 位置上，该操作的时间复杂度为 O(N)，可以看出 ArrayList 删除元素的代价是**非常高**的。

```java
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    elementData[--size] = null; // 释放引用以便原对象被垃圾回收
    return oldValue;
}
```

##### ④ **迭代与删除**

迭代器的常见误用就是在迭代的**中间**调用容器的删除方法。

```java
List<String> list = new ArrayList<>();
list.add("str1");
list.add("str2");
list.add("str3");
for (String s : list) {
    if ("str1".equals(s)) {
        // 这里使用了List提供的remove方法
        list.remove(s);
    }
}
```

这段代码看起来好像没有什么问题，但是如果我们运行，就会抛出 **ConcurrentModificationException** 异常。

其实这不是特例，每当我们使用迭代器遍历元素时，如果**修改了元素内容**（添加、删除元素），就会抛出异常，由于 **foreach** 同样使用的是迭代器，所以也有同样的情况。

remove 方法的源码

```java
public class ArrayList<E> {

    void remove() {
        modCount++;  // 继承自AbstractList的属性，保存对其中元素的修改次数，每次增加或删除时加1
        // 具体删除操作代码
        //...
    }
  
    public Iterator<E> iterator() {
        return new Itr();
    }

    private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        // 在创建迭代器时将当前ArrayList的修改次数赋值给 expectedModCount 保存
        int expectedModCount = modCount;
        public boolean hasNext() {
            return cursor != size;
        }
        
        @SuppressWarnings("unchecked")
        public E next() {
            // 检查当前所在的 ArrayList 的 modCount 是否与创建 Itr 时的值一致，
            // 也就是判断获取了Itr迭代器后 ArrayList 中的元素是否被 Itr 外部的方法改变过。
            checkForComodification();
            // 具体的获取下一个元素的代码
            // ...
        }
        
        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            // 同 next 中的 checkForComodification 方法
            checkForComodification();
            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                // Itr 内部的删除元素操作，会更新 expectedModCount 值，而外部的则不会
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
}
```

可以明显的看到共有两个`remove()`方法，一个属于 ArrayList 本身，还有一个属于其内部类 Itr。ArrayList 类中有一个 **modCount** 属性，这个属性是继承自 AbstractList，其保存了我们对 ArrayList 进行的的**操作次数**，当我们添加或者删除元素时，modeCount 都会进行对应次数的**增加**。相当于记录了**结构性变化**，即**添加、插入、删除**元素，只是修改元素的内容不算结构性变化。

在我们使用 ArrayList 的 `iterator()` 方法获取到迭代器进行遍历时，会把 ArrayList 当前状态下的 modCount 赋值给 Itr 类的 expectedModeCount 属性。如果我们在迭代过程中，使用了 ArrayList 的 `remove()`或`add()`方法，这时 modCount 就会加 1 ，但是迭代器中的expectedModeCount 并没有变化，当我们再使用迭代器的`next()`方法时，它会调用`checkForComodification()`方法，即

```java
final void checkForComodification() {
    if (modCount != expectedModCount)
    throw new ConcurrentModificationException();
}
```

发现现在的 modCount 已经与 expectedModCount **不一致**了，则会抛出`ConcurrentModificationException`异常。

上述使用了 List 提供的 remove 方法，但是如果我们使用**迭代器提供的`remove()`方法**，由于其有一个操作：`expectedModCount = modCount;`会修改 expectedModCount 的值，所以就不会存在上述问题。记得调用 remove 方法之前需要先调用 next 方法。

综上：**在单线程的遍历过程中，如果要进行 remove 操作，可以调用迭代器的 remove 方法而不是集合类的 remove 方法。**



#### 4 Fail-Fast 机制

Fail-fast 机制，即快速失败机制，是 Java **集合**(Collection)中的一种错误检测机制。当在迭代集合的过程中该集合在==**结构**上发生**改变**==的时候，就有可能会发生 fail-fast，即抛出 ConcurrentModificationException 异常。Fail-fast机制并不保证在不同步的修改下一定会抛出异常，它只是尽最大努力去抛出，所以这种机制一般仅用于检测 bug。

在我们常见的 Java 集合中就可能出现 fail-fast 机制,比如 ArrayList，HashMap。在**多线程和单线程**环境下都有可能出现快速失败。

ArrayList  中 **modCount** 用来记录 **结构发生变化的次数**。结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。

在进行**序列化或者迭代**等操作时，需要**比较**操作前后 modCount 是否改变，如果改变了需要抛出 ConcurrentModificationException。

```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

**避免fail-fast**

**方法1**

在单线程的遍历过程中，如果要进行 remove 操作，可以调用迭代器的 remove 方法而不是集合类的 remove 方法。

**方法2**

 使用并发包(java.util.concurrent)中的类来代替 ArrayList 和 HashMap。如 CopyOnWriterArrayLis t代替 ArrayList。使用 ConcurrentHashMap 替代 HashMap。





#### 5 序列化

ArrayList 基于**数组**实现，并且具有**动态扩容**特性，因此保存元素的数组不一定都会被使用，那么**就没必要全部**进行序列化。

保存元素的数组 elementData 使用 **transient** 修饰，该关键字声明数组默认**不会被序列化**。

```java
transient Object[] elementData; // non-private to simplify nested class access
```

ArrayList 实现了 writeObject() 和 readObject() 来控制==**只序列化数组中有元素填充那部分**==内容。数组没有存元素的部分不序列化。

```java
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in capacity
    s.readInt(); // ignored

    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        ensureCapacityInternal(size);

        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
```

```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

序列化时需要使用 ObjectOutputStream 的 writeObject() 将对象转换为**字节流**并输出。而 writeObject() 方法在传入的对象存在 writeObject() 的时候会去**反射**调用该对象的 writeObject() 来实现**序列化**。反序列化使用的是 ObjectInputStream 的 readObject() 方法，原理类似。

```java
ArrayList list = new ArrayList();
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
oos.writeObject(list);
```



#### 6 Arrays.asList()

Arrays 类的静态方法 asList() 将数组转为**集合**。

```java
String[] str = new String[]{"1","2","3"};
List aslist = Arrays.asList(str);
aslsit.add("4");
// Exception in thread "main" java.lang.UnsupportedOperationException
//    at java.util.AbstractList.add(Unknown Source)
//    at java.util.AbstractList.add(Unknown Source)
//    at test.LinkedListTest.main(LinkedListTest.java:13)
```

其实 asList() 返回的是 java.util.Arrays.ArrayList 对象，**不是上述的 ArrayList 类**！！！！

看 Arrays 类的部分源码

```java
public class Arrays {
    
    // 省略其他方法
 
    public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
    }
        
    // 就是这个家伙             👇
    private static class ArrayList<E> extends AbstractList<E>
            implements RandomAccess, java.io.Serializable{
    
        private final E[] a;
    
        ArrayList(E[] array) {
            a = Objects.requireNonNull(array);
        }
    
        @Override
        public int size() {
            return a.length;
        }
        //省略其他方法
    }
}
```

Arrays.ArrayList 是工具类 Arrays 的一个**内部静态类**，它**没有完全**实现 List 的方法，而 ArrayList 直接实现了List 接口，实现了 List 所有方法。Arrays.ArrayList 是一个定长集合，因为它**没有重写** add, remove 方法，所以一旦初始化元素后，集合的 size 就是不可变的。所以使用这种方法会抛 UnsupportedOperationException 异常。

**正确**的使用方式：

```java
String[] str = new String[]{"1","2","3"};
ArrayList al = new ArrayList(Arrays.asList(str)); // 将数组元素添加到集合的一种快捷方式
```









