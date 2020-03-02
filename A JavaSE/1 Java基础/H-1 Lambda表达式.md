[TOC]

### Lambda表达式

#### 0 概述

- 核心思想：用行为参数化把代码传递给方法。



#### 1 基础

- Lambda 表达式是一个**可传递的代码块**，可以在以后执行一次或多次。将一个代码块传递到某个对象 （一个定时器，或者一个sort 方法。) 这个代码块会在将来某个时间调用。

Lambda表达式**使用场景**:

- 在一个单独的线程中运行代码；
- 多次运行代码；
- 在算法的适当位置运行代码 （例如， 排序中的比较操作；)
- 发生某种情况时执行代码 （如， 点击了一个按钮， 数据到达， 等等；)
- 只在必要时才运行代码。





#### 2 语法

- lambda 表达式形式：**参数， 箭头（->) 以及一个表达式**。
- Lambda 表达式可以具有零个，一个或多个参数。
- 可以**显式声明参数**的类型，也可以由编译器**自动从上下文推断参数**的类型。
- 多个参数用**小括号**括起来，用逗号分隔。例如 `(a, b)` 或 `(int a, int b)` 或 `(String a, int b, float c)`。
- 空括号用于表示一组**空的参数**。例如 `() -> 42`。
- 当有且仅有**一个参数**时，如果不显式指明类型，则**不必使用小括号**。例如 `a -> return a * a`。
- Lambda 表达式的正文可以包含零条，一条或多条语句。
- 如果 Lambda 表达式的**正文只有一条**语句，则**大括号可不用写**，且表达式的**返回值类型**要与匿名函数的返回类型**相同**。
- 如果 Lambda 表达式的正文有一条以上的语句必须包含在大括号（代码块）中，且表达式的返回值类型要与匿名函数的返回类型相同。**必须每个分支都有返回类型**。
- 如果一个 lambda 表达式只在某些分支返回一个值， 而在另外一些分支不返回值，
    这是不合法的。 例如，（int x) -> { if (x >= 0) return 1; } 就**不合法**。

```java
// 实现一个长度比较器类
class LengthComparator implements Comparator<String> {
    public int compare(String first, String second){    // 实现接口方法
        return first.lenth - second.length();
    }
}

// 使用接口
String[] names = {"Bob", "Jack", "Alice"};
Arrays.sor(names, new LengthComparator());
```

义务代码仅一句

```java
first.lenth - second.length();
```

上述改为 Lambda 表达式

```java
// 仅一句代码
(String first, String second) 
    -> first.lenth - second.length()        // 此处无需指定返回类型，返回类型总是由上下文推断而出
    
// 代码块
(String first, String second) -> {
    // 需要return值时必须每个分支都要return值
    if (first.lengthO < second.lengthO) return -1;
    else if (first.lengthO > second.lengthO) return 1;
    else return 0;
} 
```

其他Lambda表达式

```java
// lambda 表达式没有参数， 仍然要提供空括号，就像无参数方法一样
() -> { for (int i = 100; i >= 0;i++) System.out.println(i); } 
```





#### 3 函数式接口

- 对于==只有一个抽象方法==的接口， 需要这种接口的对象时， 就可以提供一个 lambda 表达式。这种接口称为函数式接口(functional interface)。Lambda 表达式可以赋值给函数式接口。
- Arrays.sort 方法。它的第二个参数需要一个 Comparator 实例，Comparator 就是只有一个方法的接口， 所以可以提供一个lambda表达式如下。在底层， Arrays.sort方法会接收实现了Comparator\<String> 的某个类的对象。在这个对象上调用 compare 方法会执行这个 lambda 表达式的体。可以看出 lambda 表达式可以**转换**为接口。但对 lambda 表达式所能做的也只是能转换为**函数式接口**。

```java
Arrays.sort (words, (first , second) -> first.length() - second.length()) ;
```

- 如果设计的接口只有一个抽象方法，可以用 @**FunctionalInterface** 注解来标记这个接口。javadoc 页里会指出这是一个函数式接口。
- 同一个 Lambda 表达式可能可以用在**不同**的函数式接口上，只要它们的**抽象==方法签名==能够兼容**。



##### ① Supplier\<T>

```java
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

**Supplier\<T>**无参数，返回一个结果。

Supplier函数式接口为“**提供给定类型对象**”的行为提供了一种抽象。根据 Supplier 接口提供的 get 方法，我们可以实现自定义的**“提供者”**的服务。例如：

```csharp
public void test(){
  Supplier<String> supplier = () -> "welcome";
  System.out.println(supplier.get());
}
```

##### ② Consumer\<T>

```dart
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

Consumer 函数式接口为“**消费给定对象**”的行为提供了抽象，通过该函数式接口的 accept 方法可实现自定义的消费行为。例如：

```csharp
public void test(){
  Consumer<Integer> consumer = (num) -> System.out.println(num * 5);
  consumer.accept(5);
}
```

Consumer 函数式接口还提供了一个 andThen 默认方法，该默认方法提供了自身 consumer 消费行为和 after 参数提供消费行为的组合。



##### ③ Function<T,R>

```dart
@FunctionalInterface
public interface Function<T, R> {

    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }


    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

Function 函数式接口**接受一个 ==T== 类型的参数，参数一个 ==R== 类型的结果，做==类型转换==**。该函数式接口是对有一个输入参数，产生一个输出结果的一类函数或行为的抽象。例如：

```tsx
public void test(){
    Function<Integer, String> function = (age) -> "Mervyn is " + age + " years old.";
    System.out.println(function.apply(15));
}
```

Function 函数式接口还提供了 2 种组合 Function 函数式接口的默认方法：

- compose 默认方法提供了 before 参数提供的 apply 方法和自身 apply 方法的组合。将V类型参数执行 before 函数的 apply 方法后的结果作为本函数的入参，然后执行本函数的 apply 方法。
- andThen 默认方法提供了自身 apply 方法和 after 参数的 apply 方法的组合。将T类型执行自身 apply 方法的结果作为 after 参数 apply 方法的入参，然后再执行 after 参数的 apply 方法。
- Function 函数式接口还提供了一个静态方法 identify 方法用来原样返回入参的特殊 Function 类型函数。用法如下：

```csharp
public void test() {
    System.out.println(Function.identity().apply(15))
}
```



##### ④ Predicate\<T>

```dart
                           @FunctionalInterface
public interface Predicate<T> {

    boolean test(T t);

    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}
```

Predicate 函数式接口接受一个输入参数，返回 **boolean** 类型的结果，可以用于**删选**。比如筛选列表中符合条件的情况。

```java
/**
 * 自定义过滤器
 *
 * @param originList 原始列表
 * @param myFilter 自定义过滤器
 * @param <T> 类型参数
 * @return 符合条件的列表
 */
public static <T> List<T> filter(List<T> originList, Predicate<T> myFilter) {
    // 通过流的方式过滤列表中的元素
    return originList.stream().filter(myFilter).collect(Collectors.toList());
}
```

使用例子

```java
List<User> matchUser = filter(userList, user -> user.getAge > 30);
```

该函数式接口像特殊的Function<T, Boolean> 函数式接口。例如：

```csharp
public void test(){
  Predicate<Integer> predicate = (age) -> age > 15;
  System.out.println(predicate.test(16));
}
```





#### 4 方法引用

例子

```java
Timer t = new Timer(1000, event -> System.out.println(event));  // Lambda表达式
Timer t = new Timer(1000, System.out::println);     // 方法引用 两者等价
```

**System.out::println**是一个方法引用，与上述的 Lambda 表达式等价。

要用 :: 操作符分隔方法名与对象或类名。主要有 3 种情况：

- **object::instanceMethod**
- **Class::staticMethod**
- **Class::instanceMethod**

前 2 种情况中， 方法引用等价于提供方法参数的 lambda 表达式。对于第 3 种情况， 第 1 个参数会成为方法的目标。例如，String::compareToIgnoreCase 等同于 (x, y)-> x.compareToIgnoreCase(y)。

可以在方法引用中使用 this 参数。 例如， this::equals 等同于 x -> this.equals(x)。 使用
super 也是合法的。下面的方法表达式 super::instanceMethod使用 this作为目标，会调用给定方法的超类版本。

**从 Lambda 表达式到双冒号操作符**

要创建一个比较器，以下语法就足够了

```java
Comparator c = (Person p1, Person p2) -> p1.getAge().compareTo(p2.getAge());
```

然后，使用类型推断：

```java
Comparator c = (p1, p2) -> p1.getAge().compareTo(p2.getAge());
```

但是，我们可以使上面的代码更具表现力和可读性吗？我们来看一下：

```java
Comparator c = Comparator.comparing(Person::getAge);
```

使用 `::` 运算符作为 Lambda 调用特定方法的缩写，并且拥有更好的可读性。

**使用方式**

双冒号（`::`）操作符是 Java 中的**方法引用**。 当们使用一个方法的引用时，目标引用放在 `::` 之前，目标引用提供的方法名称放在 `::` 之后，即 `目标引用::方法`。比如：

```java
Person::getAge;
```

在 `Person` 类中定义的方法 `getAge` 的方法引用。

然后我们可以使用 `Function` 对象进行操作：

```java
// 获取 getAge 方法的 Function 对象
Function<Person, Integer> getAge = Person::getAge;
// 传参数调用 getAge 方法
Integer age = getAge.apply(p);
```

我们引用 `getAge`，然后将其应用于正确的参数。

目标引用的参数类型是 `Function<T,R>`，`T` 表示传入类型，`R` 表示返回类型。比如，表达式 `person -> person.getAge();`，传入参数是 `person`，返回值是 `person.getAge()`，那么方法引用 `Person::getAge` 就对应着 `Function<Person,Integer>` 类型。



#### 5 构造器引用

- 构造器引用与方法引用很类似，只不过**方法名为 new**。例如， **Person::new** 是 Person **构造**
    **器**的一个引用。调用哪个构造器取决于上下文。
- Java 有一个限制，无法构造泛型类型 T 的数组。数组构造器引用对于克服这个限制很有用。



#### 6 变量作用域

- lambda 表达式可以==捕获==外围作用域中变量的值。
- 在 lambda 表达式中， 只能引用==值不会改变==的变量。lambda表达式中捕获的变量必须实际上是**最终变量** ( ==**final 类型**==)。实际上的最终变量是指， 这个变量初始化之后就不会再为它赋新值。
- 在 Java 中， lambda 表达式就是**闭包**。



#### 7 Lambda 表达式例子

##### ① 线程初始化

线程可以初始化如下：

```java
// Old way
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello world");
    }
}).start();

// New way
new Thread(
    () -> System.out.println("Hello world")
).start();
```

##### ② 事件处理

事件处理可以用 Java 8 使用 Lambda 表达式来完成。以下代码显示了将 `ActionListener` 添加到 UI 组件的新旧方式：

```java
// Old way
button.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println("Hello world");
    }
});

// New way
button.addActionListener( (e) -> {
        System.out.println("Hello world");
});
```

##### ③ 遍例输出（方法引用）

输出给定数组的所有元素的简单代码。请注意，还有一种使用 Lambda 表达式的方式。

```java
// old way
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
for (Integer n : list) {
    System.out.println(n);
}

// 使用 -> 的 Lambda 表达式
list.forEach(n -> System.out.println(n));

// 使用方法引用
list.forEach(System.out::println);
```



##### ④ Stream API

`java.util.stream.Stream`接口 和 Lambda 表达式一样，都是 Java 8 新引入的。所有 `Stream` 的操作必须以 Lambda 表达式为参数。`Stream` 接口中带有大量有用的方法，比如 `map()` 的作用就是将 input Stream 的每个元素，映射成output Stream 的另外一个元素。

下面的例子，我们将 Lambda 表达式 `x -> x*x` 传递给 `map()` 方法，将其应用于流的所有元素。之后，我们使用 `forEach`打印列表的所有元素。

```java
// old way
List<Integer> list = Arrays.asList(1,2,3,4,5,6,7);
for(Integer n : list) {
    int x = n * n;
    System.out.println(x);
}

// new way
List<Integer> list = Arrays.asList(1,2,3,4,5,6,7);
list.stream().map((x) -> x*x).forEach(System.out::println);
```

下面的示例中，我们给定一个列表，然后求列表中每个元素的平方和。这个例子中，我们使用了 `reduce()` 方法，这个方法的主要作用是把 Stream 元素组合起来。

```java
// old way
List<Integer> list = Arrays.asList(1,2,3,4,5,6,7);
int sum = 0;
for(Integer n : list) {
    int x = n * n;
    sum = sum + x;
}
System.out.println(sum);

// new way
List<Integer> list = Arrays.asList(1,2,3,4,5,6,7);
int sum = list.stream().map(x -> x*x).reduce((x,y) -> x + y).get();
System.out.println(sum);
```



#### 8 其他

##### Lambda 表达式和匿名类之间的区别

- `this` 关键字。对于匿名类 `this` 关键字解析为匿名类，而对于 Lambda 表达式，`this` 关键字解析为包含写入 Lambda 的类。
- 编译方式。Java 编译器编译 Lambda 表达式时，会将其转换为类的私有方法，再进行动态绑定。






