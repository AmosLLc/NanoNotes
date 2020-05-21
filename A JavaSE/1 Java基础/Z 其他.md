[TOC]

### 其他

#### 1 Java编译、链接与运行

- 从源代码到运行有编译与链接两个步骤。编译将代码编译为字节码 .class 文件，一般有 **javac 命令**完成。
- 链接就是根据引用到的类加载相应的字节码并执行。
- import 是编译时概念，用于确定完全限定名，运行时只根据完全限定名寻找并加载类。

```java
javac Test.java     // Compile a class to get a .class file
java Test           // 运行上述产生的.class文件 此时不需要扩展名
```

利用 **javap** 命令对 **class 文件进行反编译**。

------

#### 2 文档注释

javadoc命令只能在/**   */之间。一些注释用的标签：

```java
@see	 	// 引用其他类
@version	// 版本号
@author		// 作者
@param		// 参数信息
@return		// 返回值信息
@throws		// 异常信息
@deprecated // 弃用
```

---

#### 3 软件开发核心原则

- **Don't Repeat Yourself:** 这是软件开发的一个基础原则，即不要做重复性劳动。也是现在所说的“极客文化”的一种。代码重复、工作重复在软件开发中都是不合理的存在。利用各种手段消除这些重复是软件开发的一个核心工作准则。
- **Keep it simple stupid**：即KISS原则。在做软件设计的工作中，很多时候都不要想得过于复杂，也不要过度设计和过早优化，用最简单且行之有效的方案也就避免了复杂方案带来的各种额外成本。既有利于后续的维护，也利于进一步的扩展。
- **You Ain’t Gonna Need It:** 即YAGNI原则。只需要将应用程序必需的功能包含进来，而不要试图添加任何其他你认为可能需要的功能。因为在一个软件中，往往80%的请求都花费在20%的功能上。
- **Done is better than perfect**: 在面对一个开发任务时，最佳的一个思路就是先把东西做出来，再去迭代优化。如果一开始就面面俱到，考虑到各种细节，那么很容易陷入牛角尖而延误项目进度。
- **Choose the most suitable things**: 这是在做方案选择、技术选型时候的一个很重要的原则。在面对许多技术方案、开源实现的时候，务必做到的是不能盲目求新，要选择最合适的而非被吹得天花乱坠的。

---

#### 4 输入输出

​     构造 Scanner 对象，并与"标准输入流 "**System.in**" 关联。

```java
import java.util.*; 
public class Test {
    public static void main(String[] args){
    Scanner s = new Scanner(System.in);     // 构建输入流
    System.out.print("Name：");
    String name = s.nextLine();             // 读取一行
    System.out.print("FirstName：");        
    String firstName  = s.next();           // 读取一个单词
    System.out.print("Age：");
    int age = s.nextInt();                  // 读取一个int类型的整数
    System.out.println("Name：" + name + " Age：" + age );
    s.close(); 		// 若没有关闭Scanner对象将会出现警告，记得关闭流
    }
}
```

牛客上输入输出总结：

```java
// 输入描述:
// 输入包括2行：
// 第一行为整数n(1 <= n <= 50)，即抹除一个数之后剩下的数字个数
// 第二行为n个整数num[i] (1 <= num[i] <= 1000000000)
import java.util.*;
public class Next {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int N = sc.nextInt();
        int[] num = new int[N];
        for (int i = 0; i < N; i++) {
            // N个数存入数组中
            num[i] = sc.nextInt();
        }
    }
}
```

```java
import java.util.*;
public class Main{
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        String str = sc.nextLine();
        char[] arr = str.toCharArray();
    }
}
```

```java
输入描述:
输入包括n+1行：
第一行为单词个数n(1 ≤ n ≤ 50)
接下来的n行，每行一个单词word[i]，长度length(1 ≤ length ≤ 50)。由小写字母构成
输出描述:
输出循环单词的种数
输入例子:
5
picture
turepic
icturep
word
ordw

Scanner sc = new Scanner(System.in);
demo.N = sc.nextInt();
demo.arr = new String[demo.N];
for (int i = 0; i < demo.N; i++) {
    String str = sc.next();
    demo.solve(str);
}
```

```java
// 读三行
// 6 3
// 1 3 5 2 5 4
// 1 1 0 1 0 0
Scanner scan = new Scanner(System.in);
// 先读第一行的两个数
int n = scan.nextInt();
int k = scan.nextInt();
// 构造两个数组
int[] val = new int[n];
int[] state = new int[n];
// 保存瞌睡时的累计评分
int sleep = 0;
int[] sleepval = new int[n];
// 读第二行N个值
for(int i=0;i<n;i++){
    val[i] = scan.nextInt();
}
// 读第三行的N个值
for(int i=0;i<n;i++){
    state[i] = scan.nextInt();
    if(state[i]==0){
        sleep += val[i];
    }
    sleepval[i] = sleep;
}
```

```java
// 第一行是告诉矩阵的行和列数
// 后面N行是矩阵
// 最后一行是需要查找的数
// 3 3
// 2 3 5
// 3 4 7
// 3 5 8
// 4
Scanner in = new Scanner(System.in);
int[][] matrix=null;
int a=0,b=0;
// 先读入第一行的两个数    
a = in.nextInt();
b = in.nextInt();
// 构造矩阵
matrix=new int[a][b];
// 读取后面的a行b列
for(int i=0;i<a;i++){
    for(int j=0;j<b;j++){
        matrix[i][j]=in.nextInt();
    }
}
// 读取最后一个数
int target=in.nextInt();
```

---

#### 5 Java 各版本的新特性

##### New highlights in Java SE 8

1. Lambda Expressions
2. Pipelines and Streams
3. Date and Time API
4. Default Methods
5. Type Annotations
6. Nashhorn JavaScript Engine
7. Concurrent Accumulators
8. Parallel operations
9. PermGen Error Removed

##### New highlights in Java SE 7

1. Strings in Switch Statement
2. Type Inference for Generic Instance Creation
3. Multiple Exception Handling
4. Support for Dynamic Languages
5. Try with Resources
6. Java nio Package
7. Binary Literals, Underscore in literals
8. Diamond Syntax

- [Difference between Java 1.8 and Java 1.7?](http://www.selfgrowth.com/articles/difference-between-java-18-and-java-17)
- [Java 8 特性](http://www.importnew.com/19345.html)

---

#### 6 Java与C++的区别

- Java 是纯粹的面向对象语言，所有的对象都继承自 java.lang.Object，C++ 为了兼容 C 即支持面向对象也支持面向过程。
- Java 通过虚拟机从而实现**跨平台**特性，但是 C++ 依赖于特定的平台。
- Java **没有指针**，它的引用可以理解为**安全指针**，而 C++ 具有和 C 一样的指针。
- Java 支持**自动垃圾回收**，而 C++ 需要手动回收。
- Java **不支持多重继承**，只能通过实现**多个接口**来达到相同目的，而 C++ 支持多重继承。
- Java **不支持操作符重载**，虽然可以对两个 String 对象执行加法运算，但是这是语言内置支持的操作，不属于操作符重载，而 C++ 可以。
- Java 的 goto 是保留字，但是不可用，C++ 可以使用 goto。
- Java **不支持条件编译**，C++ 通过 #ifdef #ifndef 等预处理命令从而实现条件编译。

---

#### 7 断言

- 断言机制允许在**测试期间**向代码中插入一些检査语句。当代码发布时，这些插人的检测
    语句将会被自动地移走。
- 关键字 assert。这个关键字有两种形式：

```java
assert 条件;
assert 条件：表达式;
```

这两种形式都会对条件进行检测， 如果结果为 false, 则抛出一个 **AssertionError** 异常。在第二种形式中，表达式将被传人 AssertionError 的构造器， 并转换成一个消息字符串。

- 默认情况下断言被**禁用**。可以在运行程序时用 -enableassertions 或 -ea 选项启用：

> java -enableassertions MyApp

- 在启用或禁用断言时**不必**重新编译程序。启用或禁用断言是**类加载器**(class loader) 的功能。当断言被禁用时，**类加载器将跳过**断言代码，因此，不会降低程序运行的速度。
- 断言失败是致命的、 不可恢复的错误。
- 断言检查只用于**开发和测试**阶段。断言只应该用于在测试阶段确定程序内部的**错误位置**。











