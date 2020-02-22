[TOC]

### Lambda表达式



#### 基础

- Lambda表达式是一个**可传递的代码块**，可以在以后执行一次或多次。将一个代码块传递到某个对象 （一个定时器，或者一个sort 方法。) 这个代码块会在将来某个时间调用。



#### 语法

- 如果一个 lambda 表达式只在某些分支返回一个值， 而在另外一些分支不返回值，
    这是不合法的。 例如，（int x)-> { if (x >= 0) return 1; } 就不合法。
- lambda 表达式形式：参数， 箭头（->) 以及一个表达式。
- Lambda 表达式可以具有零个，一个或多个参数。
- 可以显式声明参数的类型，也可以由编译器自动从上下文推断参数的类型。
- 参数用小括号括起来，用逗号分隔。例如 `(a, b)` 或 `(int a, int b)` 或 `(String a, int b, float c)`。
- 空括号用于表示一组空的参数。例如 `() -> 42`。
- 当有且仅有一个参数时，如果不显式指明类型，则不必使用小括号。例如 `a -> return a * a`。
- Lambda 表达式的正文可以包含零条，一条或多条语句。
- 如果 Lambda 表达式的正文只有一条语句，则大括号可不用写，且表达式的返回值类型要与匿名函数的返回类型相同。
- 如果 Lambda 表达式的正文有一条以上的语句必须包含在大括号（代码块）中，且表达式的返回值类型要与匿名函数的返回类型相同。



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



#### 函数式接口

- 对于==只有一个抽象方法==的接口， 需要这种接口的对象时， 就可以提供一个 lambda 表达式。这种接口称为函数式接口(functional interface)。Lambda 表达式可以赋值给函数式接口。
- Arrays.sort 方法。它的第二个参数需要一个 Comparator 实例，Comparator 就是只有一个方法的接口， 所以可以提供一个lambda表达式如下。在底层， Arrays.sort方法会接收实现了Comparator\<String> 的某个类的对象。在这个对象上调用 compare 方法会执行这个 lambda 表达式的体。可以看出 lambda 表达式可以**转换**为接口。但对 lambda 表达式所能做的也只是能转换为**函数式接口**。

```java
Arrays.sort (words, (first , second) -> first.length() - second.length()) ;
```

- 如果设计的接口只有一个抽象方法，可以用@FunctionalInterface 注解来标记这个接口。javadoc 页里会指出这是一个函数式接口。



#### 方法引用

例子

```java
Timer t = new Timer(1000, event -> System.out.println(event));  // Lambda表达式
Timer t = new Timer(1000, System.out::println);     // 方法引用 两者等价
```

**System.out::println**是一个方法引用，与上述的Lambda表达式等价。

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



#### 构造器引用

- 构造器引用与方法引用很类似，只不过**方法名为 new**。例如， Person::new 是 Person 构造
    器的一个引用。调用哪个构造器取决于上下文。
- Java 有一个限制，无法构造泛型类型 T 的数组。数组构造器引用对于克服这个限制很有用。



#### 变量作用域

- lambda 表达式可以==捕获==外围作用域中变量的值。
- 在 lambda 表达式中， 只能引用==值不会改变==的变量。lambda表达式中捕获的变量必须实际上是最终变量 ( effectivelyfinal)。实际上的最终变量是指， 这个变量初始化之后就不会再为它赋新值。
- 在 Java 中， lambda 表达式就是**闭包**。



#### 处理Lambda表达式

Lambda表达式使用场景:

- 在一个单独的线程中运行代码；
- 多次运行代码；
- 在算法的适当位置运行代码 （例如， 排序中的比较操作；)
- 发生某种情况时执行代码 （如， 点击了一个按钮， 数据到达， 等等；)
- 只在必要时才运行代码。



#### Lambda 表达式例子

**线程初始化**

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

**事件处理**

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

**遍例输出（方法引用）**

输出给定数组的所有元素的简单代码。请注意，还有一种使用 Lambda 表达式的方式。

```java
// old way
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
for (Integer n : list) {
    System.out.println(n);
}

// 使用 -> 的 Lambda 表达式
list.forEach(n -> System.out.println(n));

// 使用 :: 的 Lambda 表达式
list.forEach(System.out::println);
```

**逻辑操作**

输出通过逻辑判断的数据。

```java
package com.wuxianjiezh.demo.lambda;

import java.util.Arrays;
import java.util.List;
import java.util.function.Predicate;

public class Main {

    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7);

        System.out.print("输出所有数字：");
        evaluate(list, (n) -> true);

        System.out.print("不输出：");
        evaluate(list, (n) -> false);

        System.out.print("输出偶数：");
        evaluate(list, (n) -> n % 2 == 0);

        System.out.print("输出奇数：");
        evaluate(list, (n) -> n % 2 == 1);

        System.out.print("输出大于 5 的数字：");
        evaluate(list, (n) -> n > 5);
    }

    public static void evaluate(List<Integer> list, Predicate<Integer> predicate) {
        for (Integer n : list) {
            if (predicate.test(n)) {
                System.out.print(n + " ");
            }
        }
        System.out.println();
    }
}
```

运行结果：

```java
输出所有数字：1 2 3 4 5 6 7 
不输出：
输出偶数：2 4 6 
输出奇数：1 3 5 7 
输出大于 5 的数字：6 7 
```

**Stream API 示例**

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



#### 其他

##### Lambda 表达式和匿名类之间的区别

- `this` 关键字。对于匿名类 `this` 关键字解析为匿名类，而对于 Lambda 表达式，`this` 关键字解析为包含写入 Lambda 的类。
- 编译方式。Java 编译器编译 Lambda 表达式时，会将其转换为类的私有方法，再进行动态绑定。






