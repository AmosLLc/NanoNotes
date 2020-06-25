[TOC]

### ArrayList

#### 要点

- ArrayList 内部使用**动态数组**实现元素存储。并允许所有元素，包括 null，元素可重复。

- 实现了 RandomAccess 接口，可实现快速随机访问。

- 每次**添加元素**时都会检查动态元素数组容量是否足够，如果不够则会**动态扩容**，扩容是扩容 **1.5 倍**，扩容后会复制原来的数组，这个开销很大，所以一定要根据业务场景指定初始化容量。添加元素效率视情况而定，可能会面临数组**扩容**与内容**复制**等开销问题。指定位置的插入与删除元素效率低，因为需要**移动其它**元素。

- 动态数组是通过 **transient** 修饰的，默认不被序列化（因为动态数组可能**没有存满**），ArrayList 自定义了序列化与反序列化的方法保证只对数组中的**有效元素**进行序列化。

- ArrayList 的**迭代器**会返回一个 **内部类 Itr 对象**，**迭代时删除元素**应该使用**迭代器的 remove** 方法而非 ArrayList 本身的 remove 方法，否则会产生 **Fail Fast** 异常。当然普通使用 ArrayList 的 remove 方法是没问题的。

- **modCount** 属性是继承自 **AbstractList** 的，用来记录 **结构发生变化的次数**。结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。如果迭代或者序列化会检查 modCount 版本，如果不一致则会产生 **Fail Fast 异常**。

- ArrayList 是线程**不安全**的，建议在单线程中才使用 ArrayList，多线程可以使用 Vector 类、Collections.synchronizedList、JUC 的 **CopyOnWriteArrayList** 类等方法解决并发安全问题。

- 支持**随机访问**，按照**索引**进行访问的效率很高，效率为 O(1)。

- 按照内容**查找**元素效率**较低**，效率为 O(N)。



#### ArrayList类API

ArrayList 对象**不能存储基本类型**，只能存储**引用类型**的数据。类似 \<int> 不能写，想要存储基本类型数据，<> 中的数据类型，需要使用==基本类型包装类==，如\<Integer> 也可能用到基本数据类型，JVM会根据场景进行自动拆箱、自动装箱。

```java
public boolean add(E obj); 				// 将指定元素添加到此集合的尾部
public boolean add(int index, E obj); 	// 在指定位置插入元素，后面的元素往后移动
public E remove(int index); 			// 删除指定位置上的元素，返回被删除的元素
public E get(int index);    			// 返回指定位置上的元素
public int size();  					// 返回此集合中的元素数目
public void trimToSize();  	// 将数组列表的存储容量削减到当前尺寸，确保数组不会有新元素添加的时候调用
public void set(int index, E obj);		// 设置数组列表指定位置的值，覆盖原有内容。
```

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



#### 源码分析

下面的分析基于 **JDK8**。最新的 ArrayList 源码是有**改动**的。

因为 ArrayList 是基于**数组**实现的，所以支持**快速随机访问**。RandomAccess 接口标识着该类**支持快速随机访问**，RandomAccess 是一个**标记接口**，没有任何方法。

##### 1. 基本属性

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

实现了 **RandomAccess** 接口，代表该类支持**快速随机访问**，因为 ArrayList 是基于数组实现的。

存放元素数组为 **elementData**，其默认大小为 **DEFAULT_CAPACITY** = **10**。如果初始化时没有指定数组大小，那么第一次添加元素时会扩容，这时候就会**扩容到默认的 10**，如果指定了容量，那就按容量 1.5 扩容了。

<img src="assets/ArrayList_base-1591168031889.png" alt="1582447809913" style="zoom:71%;" />

##### 2. 初始化

主要是初始化一个指定大小的**数组**。

```java
/**
 * 不带参数的构造方法
 */
public ArrayList() {
    // 直接将空数组赋给elementData
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

/**
 * 带有容量initialCapacity的构造方法
 *
 * @param 初始容量列表的初始容量
 * @throws IllegalArgumentException 如果指定容量为负
 */
public ArrayList(int initialCapacity) {
    // 如果初始化时ArrayList大小大于0
    if (initialCapacity > 0) {
        // new一个该大小的object数组赋给elementData
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) { // 如果大小为0
        // 将空数组赋给elementData
        this.elementData = EMPTY_ELEMENTDATA;
    } else { // 小于0
        // 则抛出IllegalArgumentException异常
        throw new IllegalArgumentException("Illegal Capacity: " +
                                           initialCapacity);
    }
}
```

##### 3. 添加元素

添加元素的 add 方法如下。默认是添加到数组**最后的**位置的。这样添加还是很快的。

```java
/**
 * 添加一个值，首先会确保容量
 *
 * @param e 要添加到此列表中的元素
 * @return <tt>true</tt> (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    // 扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // 将e赋值给elementData的size+1的位置
    elementData[size++] = e;
    return true;
}
```

加元素时使用 **ensureCapacityInternal() 方法**来保证容量**足够**。

```java
/**
 * 得到最小扩容量
 *
 * @param minCapacity
 */
private void ensureCapacityInternal(int minCapacity) {
    // 调用另一个方法
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

// 首先计算capacity
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        // 如果之前为空则扩容到DEFAULT_CAPACITY为10
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

/**
 * 判断是否需要扩容
 *
 * @param minCapacity
 */
private void ensureExplicitCapacity(int minCapacity) {
    // 增加结构修改计数器
    modCount++;
    // 如果最小需要空间比elementData的内存空间要大，则需要扩容
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

如果不够时，需要使用 **grow() 方法**进行**扩容**，新容量的大小为 `oldCapacity + (oldCapacity >> 1)`，也就是旧容量的 **1.5 倍**。elementData 数组会随着实际元素个数的增多而重新分配。

```java
/**
 * 扩容，以确保它可以至少持有由参数指定的元素的数目
 *
 * @param minCapacity 当前所需的最小容量
 */
private void grow(int minCapacity) {
    // 原始数据数组长度
    int oldCapacity = elementData.length;
    // 扩容至原来的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    // 再判断一下新数组的容量够不够，够了就直接使用这个长度创建新数组，
    // 不够就将数组长度设置为需要的长度
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    // 若预设值大于默认的最大值检查是否溢出
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // 调用Arrays.copyOf方法将elementData数组指向新的内存空间时newCapacity的连续空间
    // 并将elementData的数据复制到新的内存空间
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

扩容操作需要调用 `Arrays.copyOf()` 把原数组整个复制到新数组中，这个操作**代价很高**，因此最好在创建 ArrayList 对象时就**指定**大概的容量大小，**减少扩容操作的次数**。

也可以在**指定索引**处添加元素。这里需要的操作是将 index 位置及以后的元素搬运到 index + 1之后，这个操作**开销挺大**。

```java
/**
 * 在ArrayList的index位置，添加元素element，会检查添加的位置和容量
 *
 * @param index   指定元素将被插入的索引
 * @param element 要插入的元素
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public void add(int index, E element) {
    // 判断index是否越界
    rangeCheckForAdd(index);
    // 扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    // public static void arraycopy(Object src, int srcPos, Object dest, int destPos, int length)
    // src:源数组； srcPos:源数组要复制的起始位置； dest:目的数组； destPos:目的数组放置的起始位置； length:复制的长度
    // 将elementData从index位置开始，复制到elementData的index+1开始的连续空间
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    // 在elementData的index位置赋值element
    elementData[index] = element;
    // ArrayList的大小加一
    size++;
}
```

##### 4. 获取元素

获取元素用 **get** 方法。即直接通过索引获取对应位置的元素，速度极快。

```java
public E get(int index) {
    // 检查输入的索引有没有越界
    rangeCheck(index);
	// 返回数据数组指定索引的元素
    return elementData(index);
}

E elementData(int index) {
    return (E) elementData[index];
}
```

indexOf 用于返回一个值在数组**首次出现的位置**，会根据是否为 null 使用不同方式判断。不存在就返回 -1。时间复杂度为O(N)。

```java
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i] == null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

lastIndexOf 方法效果类似，只是是反过来从后面找索引位置。

##### 5. 更新元素

更新元素依然是通过索引来的。更新之后会返回原始索引处的旧元素。

```java
public E set(int index, E element) {
    rangeCheck(index);	// 索引有效性检查
	// 获取旧元素
    E oldValue = elementData(index);
    // 设置新元素
    elementData[index] = element;
    // 返回旧元素
    return oldValue;
}
```

##### 6. 基本方法

下面是一些很基础的方法。

```java
public int size() {
    return size;
}
public boolean isEmpty() {
    return size == 0;
}
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}
```

##### 7. 序列化

ArrayList 基于**数组**实现，并且具有**动态扩容**特性，因此保存元素的数组不一定都会被使用，那么**就没必要全部**进行序列化。

保存元素的数组 **elementData** 使用 **transient** 修饰，该关键字声明数组默认**不会被序列化**。

```java
transient Object[] elementData; // non-private to simplify nested class access
```

ArrayList 实现了 writeObject() 和 readObject() 来控制==**只序列化数组中有元素填充那部分**==内容。数组没有存元素的部分不序列化。当写入到输出流时，先写入**“容量”**，再依次写入“每一个元素”；当读出输入流时，先读取“容量”，再依次读取“每一个元素”。

注意：序列化时也会**检查 modCount**，如果序列化时并发**修改**列表，可能造成 fail fast 而抛异常。

```java
/**
 * 保存数组实例的状态到一个流（即序列化）。写入过程数组被更改会抛出异常
 *
 * @serialData The length of the array backing the <tt>ArrayList</tt>
 * instance is emitted (int), followed by all of its elements
 * (each an <tt>Object</tt>) in the proper order.
 */
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException {
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    // 执行默认的反序列化/序列化过程。将当前类的非静态和非瞬态字段写入此流
    s.defaultWriteObject();

    // 写入大小
    s.writeInt(size);

    // 按顺序写入所有元素
    for (int i = 0; i < size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

```java
/**
 * 从流中重构ArrayList实例（即反序列化）。
 */
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // 执行默认的序列化/反序列化过程
    s.defaultReadObject();

    // 读入数组长度
    s.readInt(); // ignored

    if (size > 0) {
        // 像clone()方法 ，但根据大小而不是容量分配数组
        int capacity = calculateCapacity(elementData, size);
        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
        ensureCapacityInternal(size);

        Object[] a = elementData;
        //读入所有元素
        for (int i = 0; i < size; i++) {
            a[i] = s.readObject();
        }
    }
}
```

序列化时需要使用 ObjectOutputStream 的 writeObject() 将对象转换为**字节流**并输出。而 writeObject() 方法在传入的对象存在 writeObject() 的时候会去**反射**调用该对象的 writeObject() 来实现**序列化**。反序列化使用的是 ObjectInputStream 的 readObject() 方法，原理类似。

```java
ArrayList list = new ArrayList();
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
oos.writeObject(list);
```

##### 8. 删除元素

需要调用 **System.arraycopy()** 将 index + 1 后面的元素都**复制**到 index 位置上，该操作的时间复杂度为 **O(N)**，可以看出 ArrayList 删除元素的代价是**非常高**的。

注意：**remove 操作会修改 modCount 值。**

```java
public E remove(int index) {
    // 判断是否越界
    rangeCheck(index);
    
    // remove操作会修改modCount值
    modCount++;
    // 读取旧值
    E oldValue = elementData(index);
    // 获取index位置开始到最后一个位置的个数
    int numMoved = size - index - 1;
    if (numMoved > 0)
        // 将elementData数组index+1位置开始拷贝到elementData从index开始的空间
        System.arraycopy(elementData, index + 1, elementData, index,
                         numMoved);
    // 使size-1 ，设置elementData的size位置为空，让GC来清理内存空间
    elementData[--size] = null; // 便于垃圾回收器回收

    return oldValue;
}
```

##### 9. **迭代与删除**

迭代器的常见误用就是在迭代的**中间**调用容器的删除方法。

```java
List<String> list = new ArrayList<>();
list.add("str1");
list.add("str2");
list.add("str3");
for (String s : list) {
    if ("str1".equals(s)) {
        // 这里使用了List接口提供的remove方法
        list.remove(s);
    }
}
```

这段代码看起来好像没有什么问题，但是如果我们运行，就会抛出 **ConcurrentModificationException** 异常。

其实这不是特例，每当我们使用**迭代器遍历元素**时，如果**修改了元素内容**（添加、删除元素），就会**抛出异常**，由于 **foreach** 同样使用的是**迭代器**，所以也有同样的情况。

返回**迭代器**源码。

```java
public Iterator<E> iterator() {
    return new Itr();
}
```

返回的是一个**内部类 Itr** 的对象。

这个内部类如下：

```java
/**
 * 通用的迭代器实现
 */
private class Itr implements Iterator<E> {
    int cursor;       // 游标，下一个元素的索引，默认初始化为0
    int lastRet = -1; // 上次访问的元素的位置
    
    // 记录获取迭代器时的modCount，迭代过程不运行修改数组，否则就抛出异常
    int expectedModCount = modCount;

    // 是否还有下一个
    public boolean hasNext() {
        return cursor != size;
    }

    // 下一个元素
    @SuppressWarnings("unchecked")
    public E next() {
        // 检查数组是否被修改
        checkForComodification();
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i + 1;// 向后移动游标
        return (E) elementData[lastRet = i];// 设置访问的位置并返回这个值
    }

    // 删除元素
    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification(); // 检查数组是否被修改
		// 用迭代器的删除方法会自己更新modCount值
        try {
            // 这里调用ArrayList自身的remove方法，这个方法会修改modCount值
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            // 修改迭代器内部的expectedModCount为删除后当前最新的modCount值，这样就不会抛异常
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public void forEachRemaining(Consumer<? super E> consumer) {
        // 忽略....
    }

    // 检查数组是否被修改：就是判断当前列表的modCount与生成迭代器时的modCount是否一致
    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```

可以明显的看到共有两个`remove()`方法，一个属于 ArrayList 本身，还有一个属于其**内部类 Itr**。

ArrayList 类中有一个 **modCount** 属性，这个属性是继承自 AbstractList，其保存了我们对 ArrayList 进行的的**操作次数**，当我们添加或者删除元素时，modeCount 都会进行对应次数的**增加**。相当于记录了**结构性变化**，即**添加、插入、删除**元素，只是修改元素的内容不算结构性变化。

在我们使用 ArrayList 的 `iterator()` 方法获取到迭代器**进行遍历**时，会把 ArrayList 当前状态下的 **modCount** 赋值给 Itr 类的 **expectedModCount** 属性，相当于创建迭代器时候对 modCount 的一个版本快照。如果我们在迭代过程中，使用了 ArrayList 的 `remove()`或`add()`方法，这时 modCount 就会**加 1** ，但是迭代器中的 expectedModeCount **并没有**变化，当我们再使用迭代器的`next()`方法时，它会调用`checkForComodification()`方法，通过对比发现现在的 modCount 已经与 expectedModCount **不一致**了，则会抛出`ConcurrentModificationException`异常。

但是如果使用内部类 Itr ==**迭代器提供的`remove()`方法**==，它会调用 ArrayList 提供的 remove方法，同时还有一个操作：`expectedModCount = modCount;`，这会**修改**当前迭代器内部记录的 expectedModCount 的值，所以就不会存在版本不一致问题。

综上：**在单线程的遍历过程中，如果要进行 remove 操作，应该调用迭代器的 remove 方法而不是集合类的 remove 方法。**

PS：这里讨论的是**迭代删除时使用 ArrayList 的 remove 方法**，普通使用 ArrayList 的 remove 方法是没问题的。



#### Fail-Fast 机制

##### 1. 概述

通过上面的例子可以引出 Fail-fast 机制，即**快速失败机制**，是 Java **集合**(Collection)中的一种**错误检测机制**。当在**序列化或者迭代**集合的过程中该集合在==**结构**上发生**改变**==的时候，就有可能会发生 fail-fast，即抛出 **ConcurrentModificationException** 异常。

Fail-fast 机制并不保证在不同步的修改下一定会抛出异常，它只是尽最大努力去抛出，所以这种机制一般仅用于检测 bug。

在我们常见的 Java 集合中就可能出现 fail-fast 机制，比如 ArrayList，HashMap。

在**多线程和单线程**环境下都有可能出现 Fail-fast。

fail-fast 会在以下两种情况下抛出 Concurrent Modification Exception

（1）**单线程环境**

- 集合被创建后，在遍历它的过程中修改了结构。

- 注意 remove() 方法会让 expectModcount 和 modcount 相等，所以是不会抛出这个异常。

（2）**多线程环境**

- 当一个线程在遍历这个集合，而另一个线程对这个集合的结构进行了修改。

**modCount** 属性是继承自 **AbstractList** 的，用来记录 **结构发生变化的次数**。结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。可以用于**检查并发修改**的情况。

```java
protected transient int modCount = 0;
```

**modCount** 此字段由 **iterator** 和 **listiterator** 方法返回的**迭代器和列表迭代器**实现使用。**子类**是否使用此字段是**可选**的。

如果子类希望提供快速失败迭代器（和列表迭代器），则它只需在其 add(E e) 和 remove(int) 方法（以及它所重写的、导致列表结构上修改的任何其他方法）中增加此字段。

##### 2. **避免fail-fast**

**方法1**

在**单线程**的遍历过程中，如果要进行 remove 操作，应该调用**迭代器的 remove** 方法而不是集合类的 remove 方法。

**方法2**

使用并发包 (java.util.concurrent) 中的类来代替 ArrayList 和 HashMap。如 **CopyOnWriterArrayList** 代替 ArrayList。使用 **ConcurrentHashMap** 替代 HashMap。



#### Arrays.asList()

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



#### 线程安全

ArrayList 底层是以数组方式实现的，实现了**可变大小**的数组，它允许所有元素，包括 null。如果开启多个线程同时操作 List 集合，向 ArrayList 中增加元素，同时去除元素可能会出现一些问题，如数组下标越界异常。

**在多线程情况下操作ArrayList 并不是线性安全的。**

如何解决？

① 使用 **Vertor** 集合。

② 使用 **Collections.synchronizedList**。它会自动将我们的 list 方法进行改变，最后返回给我们一个加锁了的 List。

```java
protected static List<Object> arrayListSafe2 = Collections.synchronizedList(new ArrayList<Object>());  
```

③ 使用 JUC 中的 **CopyOnWriteArrayList** 类进行替换。



#### 参考资料

- [ArrayList线程安全问题](https://www.cnblogs.com/zhouyuxin/p/11154771.html)





