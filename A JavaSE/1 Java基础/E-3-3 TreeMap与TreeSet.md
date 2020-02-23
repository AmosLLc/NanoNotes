[TOC]

### TreeMap

#### 1 概述

- TreeMap 是一个**有序的 key-value 集合**，它是通过**红黑树**实现的。
- TreeMap 是按照键而不是值有序，都是对**键**进行比较。



#### 2 基本API与使用

##### ① 构造方法

- `TreeMap()`：创建一个空 TreeMap，keys 按照**自然排序**。要求 Map 中的键实现 **Comparable** 接口。

    ```java
    TreeMap<Integer, String> treeMap = new TreeMap<>();
    ```

- `TreeMap(Comparator comparator)`：创建一个空 TreeMap，按照指定的 **comparator** 排序。即创建自定义的比较器。

    ```java
    TreeMap<Integer, String> map = new TreeMap<>(Comparator.reverseOrder());
    map.put(3, "val");
    map.put(2, "val");
    map.put(1, "val");
    map.put(5, "val");
    map.put(4, "val");
    System.out.println(map); // {5 = val, 4 = val, 3 = val, 2 = val, 1 = val} 逆序
    ```

    ```java
    // String.CASE_INSENSITIVE_ORDER是String类中的一个忽略大小写的Comparator
    Map<String, String> map = new TreeMap<>(String.CASE_INSENSITIVE_ORDER);
    // 逆序并忽略大小写
    Map<String, String> map = new TreeMap<>(Collections.reverseOrder(String.CASE_INSENSITIVE_ORDER));
    ```

- `TreeMap(Map m)`：由给定的 map 创建一个 TreeMap，keys 按照自然排序。

    ```java
    Map<Integer, String> map = new HashMap<>();
    map.put(1, "val");
    ...
    TreeMap<Integer, String> treeMap = new TreeMap<>(map);
    ```

- `TreeMap(SortedMap m)`：由给定的有序 map 创建 TreeMap，keys 按照原顺序排序。



##### ② 其他方法

**增添元素**

- `V put(K key, V value)`：将指定映射放入该TreeMap中
- `V putAll(Map map)`：将指定map放入该TreeMap中

**删除元素**

- `void clear()`：清空TreeMap中的所有元素
- `V remove(Object key)`：从TreeMap中移除指定key对应的映射

**修改元素**

- `V replace(K key, V value)`：替换指定key对应的value值
- `boolean replace(K key, V oldValue, V newValue)`：当指定key的对应的value为指定值时，替换该值为新值

**查找元素**

- `boolean containsKey(Object key)`：判断该TreeMap中是否包含指定key的映射
- `boolean containsValue(Object value)`：判断该TreeMap中是否包含有关指定value的映射
- `Map.Entry<K, V> firstEntry()`：返回该TreeMap的第一个（最小的）映射
- `K firstKey()`：返回该TreeMap的第一个（最小的）映射的key
- `Map.Entry<K, V> lastEntry()`：返回该TreeMap的最后一个（最大的）映射
- `K lastKey()`：返回该TreeMap的最后一个（最大的）映射的key
- `v get(K key)`：返回指定key对应的value
- `SortedMap<K, V> headMap(K toKey)`：返回该TreeMap中严格小于指定key的映射集合
- `SortedMap<K, V> subMap(K fromKey, K toKey)`：返回该TreeMap中指定范围的映射集合（大于等于fromKey，小于toKey）

**遍历接口**

- `Set<Map<K, V>> entrySet()`：返回由该TreeMap中的所有映射组成的Set对象
- `void forEach(BiConsumer<? super K,? super V> action)`：对该TreeMap中的每一个映射执行指定操作
- `Collection<V> values()`：返回由该TreeMap中所有的values构成的集合

**其他方法**

- `Object clone()`：返回TreeMap实例的浅拷贝
- `Comparator<? super K> comparator()`：返回给该TreeMap的keys排序的comparator，若为自然排序则返回null
- `int size()`：返回该TreepMap中包含的映射的数量

```java
TreeMap<Integer, String> treeMap = new TreeMap<>();
treeMap.put(1, "a");
treeMap.put(2, "b");
treeMap.put(3, "c");
treeMap.put(4, "d"); 	// treeMap: {1 = a, 2 = b, 3 = c, 4 = d}

treeMap.remove(4); // treeMap: {1 = a, 2 = b, 3 = c}
int sizeOfTreeMap = treeMap.size(); // sizeOfTreeMap: 3

treeMap.replace(2, "e"); // treeMap: {1 = a, 2 = e, 3 = c}

Map.Entry entry = treeMap.firstEntry(); // entry: 1 -> a
Integer key = treeMap.firstKey(); // key: 1
entry = treeMap.lastEntry(); // entry: 3 -> c
key = treeMap.lastKey(); // key: 3
String value = treeMap.get(3); // value: c
SortedMap sortedMap = treeMap.headMap(2); // sortedMap: {1 = a}
sortedMap = treeMap.subMap(1, 3); // sortedMap: {1 = a, 2 = e}

Set setOfEntry = treeMap.entrySet(); // setOfEntry: [1 = a, 2 = e, 3 = c]
Collection<String> values = treeMap.values(); // values: [a, e, c]
treeMap.forEach((integer, s) -> System.out.println(integer + "->" + s)); 
// output：
// 1 -> a
// 2 -> e
// 3 -> c
```

##### ③ 遍历方式

- for循环

    ```java
    // 与HashMap迭代类似
    for (Map.Entry entry : treeMap.entrySet()) {
          System.out.println(entry);
    }
    ```

- 迭代器循环

    ```java
    Iterator iterator = treeMap.entrySet().iterator();
    while (iterator.hasNext()) {
          System.out.println(iterator.next());
    }
    ```



#### 3 源码解析

TreeMap 的基本数据结构 Entry 的部分源码，对比 HashMap 多了一些字段：

```java
static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value;
    Entry<K,V> left;
    Entry<K,V> right;
    Entry<K,V> parent;
    boolean color = BLACK;
    // ......
}
```

可以看出，Entry 中除了基本的 key、value  之外，还**有左节点、右节点以及父节点**，另外还有**颜色黑色**为 true，红色为 false。

具体看看这个：

[红黑树实现TreeMap原理解析](https://www.cnblogs.com/Joe-Go/p/10497115.html)





### TreeSet

- TreeSet 是 Set 的一个子类，TreeSet 集合是用来对象元素进行排序的，没有重复元素。实现了**排重与有序**。

- 需要传入一个 Comparator 比较器对象进行排序，或者添加的元素实现了 Comparable 接口。

- 内部基于 **TreeMap** 实现。

    





