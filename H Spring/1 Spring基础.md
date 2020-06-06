[TOC]

### Spring基础

#### 什么是Spring框架?

Spring 是一种轻量级**开发框架**，旨在提高开发人员的开发效率以及系统的可维护性。

我们一般说 Spring 框架指的都是 **Spring Framework**，它是**很多模块**的集合，使用这些模块可以很方便地协助我们进行开发。这些模块是：**核心容器、数据访问/集成,、Web、AOP（面向切面编程）、工具、消息和测试模块**。比如：Core Container 中的 Core 组件是Spring 所有组件的核心，Beans 组件和 Context 组件是实现IOC和依赖注入的基础，AOP组件用来实现面向切面编程。

Spring 官网列出的 Spring 的 6 个特征：

- **核心技术** ：依赖注入(DI)，AOP，事件(events)，资源，i18n，验证，数据绑定，类型转换，SpEL。
- **测试** ：模拟对象，TestContext框架，Spring MVC 测试，WebTestClient。
- **数据访问** ：事务，DAO支持，JDBC，ORM，编组XML。
- **Web支持** : Spring MVC和Spring WebFlux Web框架。
- **集成** ：远程处理，JMS，JCA，JMX，电子邮件，任务，调度，缓存。
- **语言** ：Kotlin，Groovy，动态语言。

#### 列举一些重要的Spring模块？

下图对应的是 Spring4.x 版本。目前最新的 5.x 版本中 Web 模块的 Portlet 组件已经被废弃掉，同时增加了用于异步响应式处理的 **WebFlux** 组件。

<img src="https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/Spring主要模块.png" alt="Spring主要模块" style="zoom: 67%;" />

- **Spring Core：** 基础,可以说 Spring 其他所有的功能都需要依赖于该类库。主要提供 **IoC 依赖注入**功能。
- **Spring  Aspects** ： 该模块为与 AspectJ 的集成提供支持。
- **Spring AOP** ：提供了**面向切面的编程**实现。
- **Spring JDBC** : Java 数据库连接。
- **Spring JMS** ：Java 消息服务。
- **Spring ORM** : 用于支持 Hibernate 等 ORM 工具。
- **Spring Web** : 为创建 Web 应用程序提供支持。
- **Spring Test** : 提供了对 JUnit 和 TestNG 测试的支持。

#### 基础注解

##### 1. @RestController VS @Controller

单独使用 `@Controller` 不加 `@ResponseBody`的话一般使用在要返回一个视图的情况，这种情况属于比较传统的Spring MVC 的应用，对应于前后端不分离的情况。

![SpringMVC 传统工作流程](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/SpringMVC传统工作流程.png)

**`@RestController` 返回JSON 或 XML 形式数据。**

但`@RestController`只返回对象，对象数据直接以 JSON 或 XML 形式写入 HTTP 响应(Response)中，这种情况属于 RESTful Web服务，这也是目前日常开发所接触的最常用的情况（前后端分离）。

![SpringMVC+RestController](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/SpringMVCRestController.png)

**`@Controller +@ResponseBody` 返回JSON 或 XML 形式数据。**

如果你需要在Spring4之前开发 RESTful Web服务的话，你需要使用`@Controller` 并结合`@ResponseBody`注解，也就是说`@Controller` +`@ResponseBody`= `@RestController`（Spring 4 之后新加的注解）。

> `@ResponseBody` 注解的作用是将 `Controller` 的方法返回的对象通过适当的转换器转换为指定的格式之后，写入到HTTP 响应(Response)对象的 body 中，通常用来返回 JSON 或者 XML 数据，返回 JSON 数据的情况比较多。

![Spring3.xMVC RESTfulWeb服务工作流程](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/Spring3.xMVCRESTfulWeb服务工作流程.png)





#### JavaBean

**JavaBean是一种组件技术**，就好像你做了一个扳子，而这个扳子会在很多地方被拿去用，这个扳子也提供多种功能(你可以拿这个扳子扳、锤、撬等等)，而这个扳子就是一个组件。

　**JavaBean是一个遵循特定写法的Java类**，它通常具有如下特点：

- 这个Java类必须具有一个无参的构造函数
- 属性必须私有化。
- 私有化的属性必须通过public类型的方法暴露给其它程序，并且方法的命名也必须遵守一定的命名规范。
- 这个类应是可序列化的。（比如可以实现Serializable 接口，用于实现bean的持久性）

许多开发者把JavaBean看作遵从特定命名约定的POJO。 简而言之，当一个POJO可序列化，有一个无参的构造函数，使用getter和setter方法来访问属性时，他就是一个JavaBean。  

```java
package gacl.javabean.study;

/**
 * @author gacl
 * Person类就是一个最简单的JavaBean
 */
public class Person {

    //Person类封装的私有属性
    // 姓名 String类型
    private String name;
    // 性别 String类型
    private String sex;
    // 年龄 int类型
    private int age;
  
   
    /**
     * 无参数构造方法
     */
    public Person() {
        
    }
    
    //Person类对外提供的用于访问私有属性的public方法
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getSex() {
        return sex;
    }
    public void setSex(String sex) {
        this.sex = sex;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
}
```

JavaBean在J2EE开发中，通常用于封装数据，对于遵循以上写法的JavaBean组件，其它程序可以通过反射技术实例化JavaBean对象，并且通过反射那些遵守命名规范的方法，从而获知JavaBean的属性，进而调用其属性保存数据。 



#### Bean

- **Bean的中文含义是“豆子”，Bean的含义是可重复使用的Java组件**。所谓组件就是一个由可以自行进行内部管理的一个或几个类所组成、外界不了解其内部信息和运行方式的群体。使用它的对象只能通过接口来操作。
- Bean并不需要继承特别的基类(BaseClass)或实现特定的接口(Interface)。Bean的编写规范使Bean的容器(Container)能够分析一个Java类文件，并将其方法(Methods)翻译成属性(Properties)，即把Java类作为一个Bean类使用。Bean的编写规范包括Bean类的构造方法、定义属性和访问方法编写规则。
- Java Bean是基于Java的组件模型，由**属性、方法和事件**3部分组成。在该模型中，JavaBean可以被修改或与其他组件结合以生成新组件或完整的程序。它是一种Java类，通过封装成为具有某种功能或者处理某个业务的对象。因此，也可以通过嵌在JSP页面内的Java代码访问Bean及其属性。



#### 传统Javabean与Spring中的bean的区别

Javabean已经没人用了

springbean可以说是javabean的发展, 但已经完全不是一回事儿了

 

**用处不同：**传统javabean更多地作为值传递参数，而spring中的bean用处几乎无处不在，任何组件都可以被称为bean。

**写法不同：**传统javabean作为值对象，要求每个属性都提供getter和setter方法；但spring中的bean只需为接受设值注入的属性提供setter方法。

**生命周期不同：**传统javabean作为值对象传递，不接受任何容器管理其生命周期；spring中的bean有spring管理其生命周期行为。

所有可以被spring容器实例化并管理的java类都可以称为bean。

原来服务器处理页面返回的值都是直接使用request对象，后来增加了javabean来管理对象，所有页面值只要是和javabean对应，就可以用类.GET属性方法来获取值。javabean不只可以传参数，也可以处理数据，相当与把一个服务器执行的类放到了页面上，使对象管理相对不那么乱（对比asp的时候所有内容都在页面上完成）。

spring中的bean，是通过配置文件、javaconfig等的设置，有spring自动实例化，用完后自动销毁的对象。让我们只需要在用的时候使用对象就可以，不用考虑如果创建类对象（这就是spring的注入）。一般是用在服务器端代码的执行上。



#### POJO

POJO 和JavaBean是我们常见的两个关键字，一般容易混淆，POJO全称是Plain Ordinary Java Object / Pure Old Java Object，中文可以翻译成：普通Java类，**具有一部分getter/setter方法的那种类就可以称作POJO**，但是JavaBean则比 POJO复杂很多， Java Bean 是可复用的组件，对 Java Bean 并没有严格的规范，理论上讲，任何一个 Java 类都可以是一个 Bean 。但通常情况下，由于 Java Bean 是被容器所创建（如 Tomcat) 的，所以 Java Bean 应具有一个无参的构造器，另外，通常 Java Bean 还要实现 Serializable 接口用于实现 Bean 的持久性。 Java Bean 是不能被跨进程访问的  

一般在web应用程序中建立一个数据库的映射对象时，我们只能称它为POJO。
POJO(Plain Old Java Object)这个名字用来强调它是一个普通java对象，而不是一个特殊的对象。
2005年11月时，“POJO”主要用来指代那些没用遵从特定的Java对象模型，约定或框架如EJB的Java对象.
理想地讲，一个POJO是一个不受任何限制的Java对象（除了Java语言规范）。例如一个POJO不应该是

      1. 扩展预定的类，如  public class Foo extends javax.servlet.http.HttpServlet { ...
    
      2. 实现预定的接口，如  public class Bar implements javax.ejb.EntityBean { ...
    
      3. 包含预定的标注，如  @javax.ejb.Entity public class Baz{ ...

  然后，因为技术上的困难及其他原因，许多兼容POJO风格的软件产品或框架事实上仍然要求使用预定的标注，譬如用于更方便的持久化。





