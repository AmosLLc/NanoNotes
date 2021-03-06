[TOC]

### 泛型

#### 基础

##### 1. 杂记

- 泛型可有泛型**类**、泛型**方法**、泛型**接口**。
- 泛型方法的返回类型之前是==**类型参数**==，可以对类型参数进行**限定**。如 **\<T extends Comparable>**。
- 通配符 ？ 代表的就是类型未知，所以限制更多。
- 编译期进行泛型**类型擦除**会把**类型变量**替换为相应的**限定类型**。
- 泛型的诸多限制多半是由**类型擦除**造成的。

##### 2. 泛型概述

使用泛型可以让编译器对**类型进行检查**，避免**插入错误类型**的对象。它提供了**编译期的类型安全**，确保你只能把**正确类型**的对象放入集合中，避免了在运行时出现 **ClassCastException**。

泛型只在**编译阶段**有效而不会进入到运行时阶段。在**编译过程**中，正确**检验泛型结果**后，会将泛型的相关信息**擦除**，并且在对象进入和离开方法的边界处**添加类型检查和类型转换**的方法。

泛型的**好处**：**安全性、可读性**。安全性是指编译器会帮检测类型错误，使类型安全；可读是指编码的时候直接就知道集合里面是什么类型。

**泛型的本质是==类型参数化==**，也就是所操作的**数据类型被指定为一个参数**。

##### 3. 泛型基本用法

###### (1) 泛型类

一个泛型类就是具有一个或多个**类型变量**的类。泛型的类型参数只能是**引用类型**，不能是基本数据类型。

```java
/*
 * 泛型类
 * Java库中E表示集合的元素类型，K和V分别表示表的关键字与值的类型
 * T（需要时还可以用临近的字母U和S）表示“任意类型”
 */
public class Pair<T> {
    // key这个成员变量的类型为T,T的类型由外部指定  
    private T first;
    private T second;

    public Pair() {first = null; second = null;}
    
    // 泛型构造方法形参key的类型也为T，T的类型由外部指定
    public Pair(T first, T second) { 
        this.first = first;  this.second = second; 
    }
	// 泛型方法getKey的返回值类型为T，T的类型由外部指定
    public T getFirst() {return first;}
    public T getSecond() {return second;}

    public void setFirst(T newValue) {first = newValue;}
    public void setSecond(T newValue) {second = newValue;}
}

// 两个类型变量的泛型类
public class Pair<T, U>{...}
```

用**具体的类型替换类型变量**就可以**实例化**泛型类型。

```java
public class PairTest1 {        // 测试类
   public static void main(String[] args)   {
      String[] words = { "Mary", "had", "a", "little", "lamb" };
      Pair<String> mm = ArrayAlg.minmax(words);
      System.out.println("min = " + mm.getFirst());
      System.out.println("max = " + mm.getSecond());
   }
}

class ArrayAlg{
   /**
    * Gets the minimum and maximum of an array of strings.
    * @param a an array of strings
    * @return a pair with the min and max value, or null if a is null or empty
    */
   public static Pair<String> minmax(String[] a) {
      if (a == null || a.length == 0) return null;
      String min = a[0];
      String max = a[0];
      for (int i = 1; i < a.length; i++)      {
         if (min.compareTo(a[i]) > 0) min = a[i];
         if (max.compareTo(a[i]) < 0) max = a[i];
      }
      return new Pair<>(min, max);
   }
}

```

定义的泛型类，就**一定要传入泛型类型实参么**？并不是这样，在使用泛型的时候如果传入泛型实参，则会根据传入的泛型实参做**相应的限制**，此时泛型才会起到本应起到的**限制作用**。如果**不传入泛型类型实参**的话，在泛型类中使用泛型的方法或成员变量定义的类型可以为**任何的类型**。

###### (2) 泛型方法

一个方法是不是泛型的，与其所在的**类**是不是泛型的**没有**关系。泛型方法也可以定义在**普通类**里面。

可以定义带有==**类型参数**==的方法。调用泛型方法时，在**方法名前的尖括号**中放人具体的类型。

**重要★**：==**\<T>**是**类型参数**，**T 是返回类型**==，类型参数放在**返回值之前**。**\<T extends Comparable>** 也是**类型参数**，限定**传入的类型**。

与泛型类不同，调用方法时一般不需要特意指定类型参数的实际类型， 一般 Java 编译器可以**自动推断**出来。

```java
// 不是泛型类
class ArrayAlg {
    
    // <T>是类型参数，T 是返回类型，参数为类型T的不定数目参数
    public static <T> T getMiddle(T... a){
        return a[a.length / 2];
    }
	
    // <T extends Comparable>是类型参数，控制传入的参数类型，T是返回类型
    public static <T extends Comparable> T min(T[] a){...}
	
    // <T>是类型参数，返回类型为int
    public static <T> int indexOf(T[] arr, T elm) {
        for(int i = 0; i < arr.length; i++) {
            if(arr[i].equals(elm)) {
                return i;
            }
        }
        return -1;
    }
}

// 调用泛型方法 注意下面的写法
String middle = ArrayAlg.<String>getMiddle("]ohnM", "Test", "Public");
// 省略类型参数
String middle = ArrayAlg.getMiddle("]ohnM", "Test", "Public");   
```

###### (3) 泛型接口

**接口**也可以是泛型的，如下。使用时需要指定**具体的类型**。

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

```java
public class Book implements Comparable<Integer> {
    public int compareTo(Integer bookPage) {
        return compare(this.bookpage, anotherBook.bookPage);
    }
}
```

##### 4. 类型参数

###### (1) 概述

泛型方法声明方式：**访问修饰符 <T,K,S...> 返回类型 方法名(方法参数) {方法体}**。访问修饰符与返回类型中间的 **<T,K,S...>** 等就是属于**类型参数**。

在泛型中，如果不对类型参数加以限制，它就可以**接受任意的**数据类型，只要它是被定义过的。但很多时候我们只需要一部分数据类型就够了，用户传递其他数据类型可能会引起错误。**类型参数**能够限定类型**必须实现某些接口或者继承某个类**。

###### (2) 基本使用

类型参数上界可以是**类或接口**，此时 **T** 必须**实现**这个接口或者继承这个类。如果有多个类型变量和多个限定类型，限定类型用 **& 分隔**，类型变量用**逗号分隔**。

```java
// 限定min方法只能被实现了Comparab1e接口的类调用
public static <T extends Comparable> T min(T[] a){...}
```

如果用一个**类作为限定**，它必须是限定列表中的**第一个**。

```java
// 多个限定类型，且Students类为第一个
public static <T extends Students & Comparab1e & Serializable> Pair<T> min(T[] a){...}
```

```java
/**
 * 泛型方法 <T extends Comparable> 为类型限定参数  Pair<T> 为返回类型  T[]为入参
 */
public static <T extends Comparable> Pair<T> minmax(T[] a) {
      if (a == null || a.length == 0) return null;
      T min = a[0];
      T max = a[0];
      for (int i = 1; i < a.length; i++) {
         if (min.compareTo(a[i]) > 0) min = a[i];
         if (max.compareTo(a[i]) < 0) max = a[i];
      }
      return new Pair<>(min, max);
   }
```



#### 通配符

##### 1. 概述

除了用 \<T> 表示泛型外，还有 **\<?>** 这种形式。**？** 被称为**通配符**。既然已经有了 \<T> 的形式了，为什么还要引进 \<?> 这样的概念呢？在现实编码中可能希望泛型能够**处理某一范围内的数据类型**，比如**某个类和它的子类**，对此 Java 引入了通配符这个概念。**通配符的出现是为了指定泛型中的类型范围**。

举个栗子：

```java
class Base{}	// 父类
class Sub extends Base{}	// 子类

Sub sub = new Sub();
Base base = sub;
```

上面代码显示，Base 是 Sub 的**父类**，它们之间是**继承关系**，所以 Sub 的实例可以给一个 Base 引用赋值，但是：

```java
List<Sub> lsub = new ArrayList<>();
List<Base> lbase = lsub;
```

上述第二行是不行的，**编译通不过**。Sub 是 Base 的子类，**不代表 List\<Sub> 和 List\<Base> 有继承关系**。Java 中**集合是==不能协变==**的，也就是说 List\<Base> 不是 List\<Sub> 的父类，这时候就可以用到**通配符**了。

##### **2. 通配符分类**

通配符主要有以下三类：

- **无边界的通配符**：就是 ==**\<?>**==, 比如 **List<?>**。无边界的通配符的主要作用就是让**泛型能够接受==未知类型==**的数据。
- **固定上边界**的通配符：使用**固定上边界**的通配符的泛型, 就能够接受**指定类及其子类类型**的数据。要声明使用该类通配符, 采用 ==**<? extends E> **== 的形式，这里的 **E** 就是该泛型的**上边界**。注意：这里虽然用的是 **extends** 关键字, 却不仅限于继承了父类 E 的子类, 也可以代指实现了**接口** E 的**类**。
- **固定下边界**的通配符：使用**固定下边界**的通配符的泛型, 就能够接受**指定类及其父类类型**的数据。要声明使用该类通配符, 采用 ==**<? super E>**== 的形式, 这里的 **E** 就是该泛型的**下边界**。

注意：你可以为一个泛型指定上边界或下边界, 但是**不能同时**指定上下边界。

##### 3. 基本使用方法

###### (1) 无边界通配符的使用

以在集合 List 中使用 **<?>** 为例。

```java
public static void printList(List<?> list) {
    for (Object o : list) {
        // 这里只是读取集合元素
        System.out.println(o);
    }
}

public static void main(String[] args) {
    List<String> l1 = new ArrayList<>();
    l1.add("aa");
    l1.add("bb");
    l1.add("cc");
    printList(l1);
    List<Integer> l2 = new ArrayList<>();
    l2.add(11);
    l2.add(22);
    l2.add(33);
    printList(l2);
}
```

这种使用 **List<?>** 的方式就是**父类引用指向子类对象**。注意，这里的 printList 方法**不能写成** public static void printList(List\<**Object**> list) 的形式, 虽然 Object 类是所有类的父类，但是 List\<Object> 跟其他泛型的 List 如 List\<String>，List\<Integer> **不存在**继承关系，因此会报错。

**问号  ？ 表示类型安全未知，即只能读不能写**。

所以**不能对 List<?> 使用 add 方法（单使用 ？只读，不能修改了）, 仅有一个例外, 就是 add(null)**。因为我们**不确定**该 List 的类型，不知道 add 什么类型的数据才对，只有 null 是所有引用数据类型都**具有**的元素。

请看下面代码：

```java
public static void addTest(List<?> list) {
    Object o = new Object();
    // list.add(o); // 编译报错
    // list.add(1); // 编译报错
    // list.add("ABC"); // 编译报错
    list.add(null);
}
```

还有，**List<?> 也不能使用 get 方法, 只有 Object 类型是个例外**。原因也很简单, 因为我们不知道传入的 List 是什么泛型的, 所以无法接受得到的 get, 但是 **Object** 是所有数据类型的父类，所以可以用于接受数据：

```java
public static void getTest(List<?> list) {
    // String s = list.get(0);  // 编译报错
    // Integer i = list.get(1); // 编译报错
    Object o = list.get(2);
}
```

不是有**强制类型转换**么? 但是我们根本**不知道**会传入什么类型，比如我们将其强转为 String，编译是通过了, 但是如果传入个 Integer 泛型的 List，一运行还会出错。那么保证传入的 String 类型的数据不就好了么? 那样是没问题了, 但是那还用 <?> 干嘛? 直接 List\<String> 就行了。

###### (2) 固定上边界通配符的使用

 仍以 List 为例来说明。它失去了**写操作**的能力。

```java
public static double sumOfList(List<? extends Number> list) {
    double s = 0.0;
    for (Number n : list) {
        // 注意这里得到的n是其上边界类型的, 也就是Number, 需要将其转换为double.
        s += n.doubleValue();
    }
    return s;
}

public static void main(String[] args) {
    List<Integer> list1 = Arrays.asList(1, 2, 3, 4);
    System.out.println(sumOfList(list1));
    List<Double> list2 = Arrays.asList(1.1, 2.2, 3.3, 4.4);
    System.out.println(sumOfList(list2));
}
```

**List<? extends E> 不能使用 add 方法（也就是前面的例子中 extends 只读不能修改！）**。

```java
public static void addTest2(List<? extends Number> l) {
    // l.add(1);   // 编译报错
    // l.add(1.1); // 编译报错
    l.add(null);
}
```

原因很简单，泛型 **<? extends E>** 指的是 **E 及其子类**，这里传入的可能是 Integer，也可能是 Double，但是在写这个方法时**不能确定传入什么类型的数据**，如果我们调用：

```java
List<Integer> list = new ArrayList<>();
addTest(list);
```

那么我们之前写的 add(1.1) 就会出错，反之亦然，所以**除了 null 之外什么也不能 add**。==但是 **get 的时候是可以得到一个 Number**，也就是**上边界类型**的数据的，因为不管存入什么数据类型都是 Number 的**子类型**==，得到这些就是一个**父类引用指向子类**对象。有点东西，这就是**上边界**的作用。

###### ③ 固定下边界通配符的使用

这个较前面的两个有点难理解，仍以 List 为例。它拥有**一定程度的写操作**的能力。

```java
public static void addNumbers(List<? super Integer> list) {
    for (int i = 1; i <= 10; i++) {
        list.add(i);
    }
}

public static void main(String[] args) {
    List<Object> list1 = new ArrayList<>();
    addNumbers(list1);
    System.out.println(list1);
    List<Number> list2 = new ArrayList<>();
    addNumbers(list2);
    System.out.println(list2);
    List<Double> list3 = new ArrayList<>();
    // addNumbers(list3); // 编译报错
}
```

**List<? super E>** 是能够调用 **add 方法**的，因为我们在 addNumbers 方法中所 **add 的元素就是 Integer 类型的**，而传入的 list 不管是什么，都一定是 Integer 或其父类泛型的 List，这时 add 一个 Integer 元素是没有任何疑问的。但是，**我们==不能使用 get 方法==**，除非使用 **Object** 类型来接收。

```java
public static void getTest2(List<? super Integer> list) {
    // Integer i = list.get(0); // 编译报错
    Object o = list.get(1);
}
```

这是因为我们所传入的类都是 **Integer 的类或其父类**，所**传入**的数据类型可能是 **Integer 到 Object** 之间的**任何类型**，这是**无法预料**的，也就无法接收。唯一能确定的就是 **Object**, 因为所有类型都是其**子类型**。
使用 **<? super E>** 还有个常见的场景就是 **Comparator**。

###### (4) 类型参数与通配符的使用条件

一般而言，**通配符**能干的事情都可以用**类型参数替换**。 比如：

```java
public void testWildCards(Collection<?> collection){}
```

可以换成：

```java
public <T> void test(Collection<T> collection){}
```

如果用**泛型方法**来取代通配符，那么上面代码中 collection 是**能够进行写操作**的。只不过要进行**强制转换**。泛型方法中的泛型参数对象是**可修改**的，因为**类型参数 T 是确定**的（在调用方法时确定），因为 T 可以用范围内任意类型指定。但是通配符本身就**代表类型未知**，所以可能出现**无法修改**或者**只读**的情况，限制更多。

此外，**类型参数**适用于**参数之间的类别依赖关系**，举例说明，比如下面 **E T 两个**参数之间是有**依赖关系**的时候。E 类型是 T 类型的子类，显然这种情况**类型参数**更适合。 

```java
public class Test2 <T, E extends T>{
   T value1;
   E value2;
}
```

```java
public <D, S extends D> void test(D d, S s){
}
```

如果一个方法的**返回类型**依赖于**参数的类型**，那么通配符也无能为力。



#### 类型擦除

##### 1. 概述

Java 泛型是通过**类型擦除**实现的。 **类型擦除**指的是**泛型相关的信息在编译后被擦除的情况**，其过程就是**擦除(erased) 类型变量**, 并**替换**为相应的**限定类型**。**类型参数**给类型擦除指定一个**边界**，类型擦除之后所有的类型参数都用它们的**限定类型替换**，无限定类型的变量用 **Object** 替换。所以泛型信息只存在于代码**编译阶段**。

虚拟机**没有**泛型类型对象，虚拟机不知道泛型，所有对象都属于**普通类**，泛型类和普通类在 Java 虚拟机内是没有什么特别的地方。

###### (1) 没有限定类型参数

自定义一个**泛型类**。

```java
public class Erasure<T> {
    T object;

    public Erasure(T object) {
        this.object = object;
    }
}
```

我们通过**反射查看它在运行时的状态信息**。

```java
Erasure<String> erasure = new Erasure<String>("hello");
Class erasureClass = erasure.getClass();
System.out.println("Erasure class is:" + erasureClass.getName());
```

**输出**

```java
Erasure class is:javase.genericity.Erasure
```

Class 的**类型**是 **Erasure** 而不是 Erasure\<T> 的形式，再用反射看看泛型类中 **T 的类型**在 **JVM** 中具体是什么类型。

```java
Field[] fs = erasureClass.getDeclaredFields();
for (Field f : fs) {
    System.out.println("Field name: " + f.getName() + 
                       ", type:" + f.getType().getName());
}
```

```java
Field name: object, type:java.lang.Object
```

由于**没有限定类型**，所以 T 换为 ==**Object**== 类型。

###### (2) 有限定类型参数

再给**类型参数**加上更多的限定如下。

```java
public class Erasure <T extends String> {
    T object;

    public Erasure(T object) {
        this.object = object;
    }
}
```

可以看到 T 类型替换成了**类型上限 String**。

```java
Field name: object, type:java.lang.String
```

所以：在泛型类被**类型擦除**的时候，如果类型参数**没有指定上限**，如 \<T> 则会被转译成普通的 **Object** 类型，如果指定了类型参数上限如 \<T extends String> 则类型参数就被**替换成类型上限**。

##### 2. 泛型的约束与局限性

由于**类型擦除**机制的存在，会引起诸多泛型使用的**约束与局限性**。

**1. 不能用基本类型实例化类型参数**

不能用类型参数代替基本类型。因此没有 Pair\<double>, 只有 **Pair\<Double>**。其原因是**类型擦除**。擦除之后，Pair 类含有 **Object** 类型的域，而 Object 不能存储 double 值。

**2. 运行时类型查询只适用于==原始类型==**

虚拟机中的对象总有一个特定的非泛型类型。因此 **getClass** 方法查询得到的是**原始**类型。

```java
Pair<String> stringPair = . .
Pair<Employee> employeePair = . .
if (stringPair.getClass() == employeePair.getClass()) // true
```

其比较的结果是 true, 这是因为两次调用 getClass 都将返回其**原始**类型 **Pair.class**。

**3. 不能通过类型参数创建对象**

**不能**使用像 **new T(...)  new T[...]  或  T.class**  这样的表达式中的类型变量。最好的是让调用者提供一个**构造器**表达式。

```java
// 非法
public Pair() { 
    first = new T(); 
    second = new T(); 
} 

Pair<String> p = Pair.makePair(String::new);    // 提供构造表达式：合法
```

如果需要**根据类型创建对象**，可以通过**反射**的方式实现。

```java
// 非法 T.class非法，会转为Object.class
first = T.class.newInstance(); 
// 可以通过以下的API得到class对象
public static <T> Pair<T> makePair(Class<T> cl){
    try { 
        return new Pair<>(cl.newInstance(), cl.newInstance()); }
    catch (Exception ex) { 
        return null; 
    }
}
// 使用如下方式调用
Pair<String> p = Pair.makePair(String.class);
```

**4. 不能创建具体类型的数组**

不能实例化参数化类型的**数组**， 例如：

```java
Pair<String>[] table = new Pair<String>[10]; // 错误 传入String已经是参数化了
List<Integer>[] li2 = new ArrayList<Integer>[];
List<Boolean>[] li3 = new ArrayList<Boolean>[];
```

这三行代码是**无法**在编译器中编译通过的。原因还是**类型擦除**带来的影响。擦除之后，table 的**类型是 Pair[]**。 可以把它转换为 Object[]：

```java
Object[] objarray = table;
```

而 List\<Integer> 和 List\<Boolean> 在 JVM 中等同于 **List\<Object>** ，所有的类型信息**都被擦除**，程序也无法分辨一个数组中的元素**类型具体**是 List\<Integer>类型还是 List\<Boolean> 类型。

数组会**记住**它的**元素类型**， 如果试图存储其他类型的元素，就会抛出一个 ArrayStoreException 异常。

**但是**可以向**参数个数可变的方法**传递一个泛型类型的实例。如下面的参数可变的方法:

```java
public static <T> void addAll(Collections coll, T... ts){
    for (t : ts) coll.add(t);
}
```

为了调用这个方法，Java 虚拟机必须建立一个 Pair\<String> 数组，这就违反了前面的规则。不过这种情况只会得到一个**警告**，而不是错误。

**5. 不能创建泛型数组**

**数组本身也有类型**，用来监控存储在虚拟机中的数组。这个类型会被**擦除**。

```java
public static <T extends Comparable> T[] minmax(T[] a) { 
    // 非法
    T[] mm = new T[2]; 
} 
```

类型擦除会让这个方法**永远构造 Comparable[2] 数组**。如果现实需要能够存放**泛型对象**的容器，可以使用**原始类型**的数组，如。

```java
Pair[] options = new Pair {
    new Pair<String, Integer>("1", 2),
    new Pair<String, Integer>("2", 2)
};
```

泛型容器内部使用 **Object 数组**，如果要转换泛型容器为对应类型的数组，需要使用反射。

最好让用户提供一个**数组构造器表达式**：

```java
String[] ss = ArrayAlg.minmax(String[]::new, "Tom", "Dick", "Harry");
```



#### 面试题

> **Java的泛型是如何工作的 ? 什么是类型擦除 ?**

泛型是通过**类型擦除**来实现的，编译器在**编译**时**擦除了所有类型相关的信息**，所以在**运行时不存在任何类型相关的信息**。

>  **什么是泛型中的限定通配符和非限定通配符 ?**

这是一个非常流行的面试题。限定通配符对**类型**进行了限制。有两种**限定通配符**，一种是 **\<? extends T>** 它通过确保类型必须是 T 的子类来设定类型的**上界**，另一种是 **\<? super T>** 它通过确保类型必须是 T 的父类来设定类型的**下界**。泛型类型必须用限定内的类型来进行初始化，否则会导致编译错误。还有一个 **\<?>** 表示了**非限定通配符**，因为 <?> 可以用**任意类型**来替代。

> **List<? extends T>和List <? super T>之间有什么区别 ?**

这两个 List 的声明都是使用了**限定通配符**，List<? extends T> 可以**接受任何继承自 T 的类型**的 List，而 List<? super T> 可以**接受任何 T 的父类构成的 List**。例如 List<? extends Number> 可以接受 List\<Integer> 或 List\<Float>。

> **你可以把List\<String>传递给一个接受List\<Object>参数的方法吗？**

因为乍看起来 String 是一种 Object，所以 List\<String> 应当可以用在需要 List\<Object>的地方，但是**事实并非如此**。这就是前面提到的集合是**不能协变**的。真这样做的话会导致**编译错误**。　

```java
List<Object> objectList;
List<String> stringList;
objectList = stringList;  // compilation error incompatible types
```

> **Array中可以用泛型吗?**

Array 事实上**并不支持泛型**，这也是为什么 Effective Java 一书中建议使用 List 来代替 Array，因为 **List 可以提供编译期的类型安全保证，而 Array 却不能**。





#### 参考资料

- [通配符详解](https://blog.csdn.net/sinat_27143551/article/details/80985477)
- [详细分析通配符与类型参数的区别](https://blog.csdn.net/sinat_32023305/article/details/83215751)

- [深入理解泛型](https://blog.csdn.net/sinat_27143551/article/details/80985477)
- [从字节码的角度看泛型](https://blog.csdn.net/zl1zl2zl3/article/details/83301799)









