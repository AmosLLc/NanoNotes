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