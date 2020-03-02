[TOC]

### 流 Stream

#### 1 概述

什么是流？
Stream **不是集合**元素，它不是数据结构并不保存数据，它是有关算法和计算的，它更像一个高级版本的 **Iterator**。原始版本的 Iterator，用户只能显式地一个一个遍历元素并对其执行某些操作；高级版本的 Stream，用户只要给出需要对其包含的元素执行什么操作，比如 “过滤掉长度大于 10 的字符串”、“获取每个字符串的首字母”等，Stream 会**隐式地在内部进行遍历**，做出相应的数据转换。

流可以在**内部进行迭代**，简化代码。

Stream 就如同一个迭代器（Iterator），单向，不可往复，数据只能遍历一次，遍历过一次后即用尽了，就好比流水从面前流过，一去不复返。

而和迭代器又不同的是，Stream 可以**并行化**操作，迭代器只能命令式地、串行化操作。顾名思义，当使用串行方式去遍历时，每个 item 读完后再读下一个 item。而使用并行去遍历时，数据会被分成多个段，其中每一个都在不同的线程中处理，然后将结果一起输出。Stream 的并行操作依赖于 Java7 中引入的 Fork/Join 框架（JSR166y）来拆分任务和加速处理过程。

Stream 的另外一大特点是，数据源本身可以是无限的。


#### 2 产生流

获取一个数据源（source）→ 数据转换→执行操作获取想要的结果。每次转换原有 Stream 对象不改变，返回一个新的 Stream 对象（可以有多次转换），这就允许对其操作可以像链条一样排列，变成一个管道。

有多种方式生成 Stream Source：

- 从 Collection 和数组

```java
Collection.stream()
Collection.parallelStream()
Arrays.stream(T array) or Stream.of()
```

- 从 BufferedReader

```java
java.io.BufferedReader.lines()
```

- 静态工厂

```java
java.util.stream.IntStream.range()
java.nio.file.Files.walk()
```

- 自己构建

```java
java.util.Spliterator
```

- 其它

```java
Random.ints()
BitSet.stream()
Pattern.splitAsStream(java.lang.CharSequence)
JarFile.stream()
```



#### 3 使用示例

```java
List<Dish> menu = ...

// 通过集合对象产生流
List<String> lowCaloricDishesName = menu.stream()
    // 筛选出卡路里大于400的
    .filter(d -> d.getCalories() < 400)
    // 抽取名字属性创建一个新的流
    .map(Dish::getName)
    // 这个流按List类型返回
    .collect(toList());
```

在这段代码 **filter** 和 **map** 操作被称为**中间操作**,中间操作会**返回一个新的流**，而 **collect** 则被称为**终端操作**只有终端操作才会让整个流执行并关闭。也就是说 **每个流只能遍历一次** ，因为 collect 以后这个流就已经**关闭**了。

中间可以做**类型转换、筛选、规约、统计**等操作。



#### 4 性能分析

对于简单操作，比如最简单的遍历，Stream串行API性能明显差于显示迭代，但并行的Stream API能够发挥多核特性。

如果出于**性能考虑**，1. 对于简单操作推荐使用外部迭代手动实现，2. 对于复杂操作，推荐使用 Stream API， 3. 在多核情况下，推荐使用并行 Stream API 来发挥多核优势，4.单核情况下不建议使用并行 Stream API。

如果出于**代码简洁性考虑**，使用 Stream API 能够写出更短的代码。即使是从性能方面说，尽可能的使用 Stream API 也另外一个优势，那就是只要 Java Stream 类库做了升级优化，代码不用做任何修改就能享受到升级带来的好处。









