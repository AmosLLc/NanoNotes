[TOC]



### ArrayList

#### 1. 概览

- 使用时需要实例化泛型参数。
- 不是线程安全的。
- 可以随机访问，按照索引进行访问的效率很高，效率为 O(1)。
- 除非数组已经排序，否则按照内容查找元素效率较低，效率为 O(N)。
- 添加元素效率还行，但是会面临数组扩容与内容复制等开销问题。
- 插入与删除元素效率较低，因为需要移动元素。



因为 ArrayList 是基于**数组**实现的，所以支持**快速随机访问**。RandomAccess 接口标识着该类支持快速随机访问，RandomAccess 是一个标记接口，没有任何方法。

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

数组的默认大小为 10。

```java
private static final int DEFAULT_CAPACITY = 10;
```

![1563604782084](assets/1563604782084.png)



#### 2. 扩容

添加元素时使用 ensureCapacityInternal() 方法来保证容量足够，如果不够时，需要使用 grow() 方法进行扩容，新容量的大小为 `oldCapacity + (oldCapacity >> 1)`，也就是旧容量的 **1.5 倍**。elementData 数组会随着实际元素个数的增多而重新分配。

扩容操作需要调用 `Arrays.copyOf()` 把原数组整个复制到新数组中，这个操作代价很高，因此最好在创建 ArrayList 对象时就**指定**大概的容量大小，**减少扩容操作的次数**。

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;		// size 用于记录实际的元素个数	
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

#### 3. 删除元素

需要调用 System.arraycopy() 将 index+1 后面的元素都复制到 index 位置上，该操作的时间复杂度为 O(N)，可以看出 ArrayList 删除元素的代价是**非常高**的。

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

**迭代与删除**

迭代器的常见误用就是在迭代的中间调用容器的删除方法。

```java
List<String> list = new ArrayList<>();
list.add("str1");
list.add("str2");
list.add("str3");
for (String s : list) {
    if ("str1".equals(s)) {
        list.remove(s);
    }
}
```

这段代码看起来好像没有什么问题，但是如果我们运行，就会抛出 **ConcurrentModificationException** 异常。

其实这不是特例，每当我们使用迭代器遍历元素时，如果修改了元素内容（添加、删除元素），就会抛出异常，由于 **foreach** 同样使用的是迭代器，所以也有同样的情况。

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

可以明显的看到共有两个`remove()`方法，一个属于 ArrayList 本身，还有一个属于其内部类 Itr。ArrayList 类中有一个 modCount 属性，这个属性是继承子AbstractList，其保存了我们对 ArrayList 进行的的**操作次数**，当我们添加或者删除元素时，modeCount 都会进行对应次数的**增加**。相当于记录了结构性变化，即添加、插入、删除元素，只是修改元素的内容不算结构性变化。

在我们使用 ArrayList 的 `iterator()` 方法获取到迭代器进行遍历时，会把 ArrayList 当前状态下的 modCount 赋值给 Itr 类的 expectedModeCount 属性。如果我们在迭代过程中，使用了 ArrayList 的 `remove()`或`add()`方法，这时 modCount 就会加 1 ，但是迭代器中的expectedModeCount 并没有变化，当我们再使用迭代器的`next()`方法时，它会调用`checkForComodification()`方法，即

```java
final void checkForComodification() {
    if (modCount != expectedModCount)
    throw new ConcurrentModificationException();
}
```

发现现在的 modCount 已经与 expectedModCount **不一致**了，则会抛出`ConcurrentModificationException`异常。

但是如果我们使用迭代器提供的`remove()`方法，由于其有一个操作：`expectedModCount = modCount;`，会修改expectedModCount 的值，所以就不会存在上述问题。调用remove 方法之前需要先调用 next 方法。



#### 4. Fail-Fast

fail-fast 机制在遍历一个集合时，当集合结构被修改，会抛出 ConcurrentModificationException。

modCount 用来记录 ArrayList **结构发生变化的次数**。结构发生变化是指添加或者删除至少一个元素的所有操作，或者是调整内部数组的大小，仅仅只是设置元素的值不算结构发生变化。

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



#### 5. 序列化

ArrayList 基于数组实现，并且具有动态扩容特性，因此保存元素的数组不一定都会被使用，那么**就没必要全部**进行序列化。

保存元素的数组 elementData 使用 transient 修饰，该关键字声明数组默认**不会被序列化**。

```java
transient Object[] elementData; // non-private to simplify nested class access
```

ArrayList 实现了 writeObject() 和 readObject() 来控制**只序列化数组中有元素填充那部分**内容。数组没有存元素的部分不序列化。

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

#### 6. Arrays.asList()

Arrays类的静态方法 asList() 将数组转为集合。

```java
String[] str = new String[]{"1","2","3"};
List aslist = Arrays.asList(str);
aslsit.add("4");
// Exception in thread "main" java.lang.UnsupportedOperationException
//    at java.util.AbstractList.add(Unknown Source)
//    at java.util.AbstractList.add(Unknown Source)
//    at test.LinkedListTest.main(LinkedListTest.java:13)
```

其实 asList() 返回的是 java.util.Arrays.ArrayList 对象，不是上述的 ArrayList 类！！！！

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









