[TOC]

## Spring IoC

### Spring Bean

在 Spring 中，那些组成应用程序的主体及由 Spring IOC 容器所管理的对象，被称之为 bean。简单地讲，bean 就是由 IOC 容器初始化、装配及管理的对象，除此之外，bean 就与应用程序中的其他对象没有什么区别了。而 bean 的定义以及 bean 相互间的依赖关系将通过配置元数据来描述。

#### Bean Scope

Scope 用来声明容器中的对象所应该处的限定场景或者说该对象的存活时间，即容器在对象进入其相应的 scope之前，生成并装配这些对象，在该对象不再处于这些 scope 的限定之后，容器通常会销毁 Spring 的 IoC 容器这些对象。

如果你不指定 bean 的 scope，**singleton** 便是容器**默认**的 scope.

- **singleton**: 在 Spring 的 IoC 容器中只存在**一个实例**，所有对该对象的引用将**共享**这个实例。该实例从容器启动，并因为第一次被请求而初始化之后，将一直存活到容器退出，也就是说，它与 IoC 容器“几乎”拥有相同的“寿命”。标记为 singleton 的 bean是由容器来保证这种类型的 bean 在**同一个容器中**只存在一个共享实例；而 Singleton 模式则是保证在同一个 **Classloader** **中**只存在一个这种类型的实例。

- **propertype**: 针对声明为拥有 prototype scope 的 bean 定义，容器在接到该类型对象的请求的时候，会每次都**重新生成一个新的对象实例**给请求方。虽然这种类型的对象的实例化以及属性设置等工作都是由容器负责的，但是只要准备完毕，并且对象实例返回给请求方之后，**容器就不再拥有当前返回对象的引用**，请求方需要自己负责当前返回对象的后继生命周期的管理工作，包括该对象的销毁。
    对于那些请求方不能共享使用的对象类型，应该将其 bea n定义的 scope 设置为 prototype。这样每个请求方可以得到自己对应的一个对象实例。通常，声明为 prototype 的 scope 的 bean 定义类型，都是==**一些有状态的**==，比如保存每个顾客信息的对象。

另外三种 scope 类型，即 request、session 和 global session 类型,只有在支持 Web 应用的 **ApplicationContext **中才能使用这三个 scope。

- **request**: 为每一个 **HTTP** 请求创建一个全新的 bean，当请求结束之后，该对象的生命周期即告结束。
- **session**: 为每一个独立的 **session** 创建一个全新的 bean 对象，session 结束之后，该 bean 对象的生命周期即告结束。
- **global session**: global session只有应用在基于 portlet的Web应用程序中才有意义，它映射到portlet的global范围的 session。用得少。

注意：singleton 的对象在 IOC 容器创建的时候就会创建。在不指定 @Scope 的情况下，所有的 bean 都是**单实例**的 bean, 而且是**饿汉加载**（容器启动实例就创建好了）。

指定 @Scope 为 **prototype** 表示为**多实例**的，而且还是**懒汉模式**加载（IOC 容器启动的时候，并不会创建对象，而是在第一次使用的时候才会创建。

如果对 singleton 的 bean 指定为 **@Lazy 懒加载**，那么会在第一次使用时创建。Bean 的懒加载 @Lazy (主要针对**单实例**的bean 容器启动的时候，不创建对象，在第一次使用的时候才会创建该对象)  。

````java
@Bean
@Lazy
public Person person() {
    return new Person();
}
````






#### Bean生命周期

Spring 容器将对其所管理的对象全部给予**统一的生命周期**管理，这些被管理的对象完全摆脱了原来那种“new 完后被使用，脱离作用域后即被回收”的命运。

 ![1573540627894](assets/1573540627894.png) 

**==什么时候创建Bean？==**

容器启动之后，并不会马上就实例化相应的 bean 定义。容器现在仅仅拥有所有对象的 **BeanDefinition** 来保存**实例化**阶段将要用的必要信息。只有当请求方通过 **BeanFactory** 的 **getBean**() 方法来请求某个**对象实例**的时候，才**有可能触发** Bean实例化阶段的活动。BeanFactory 的 getBean() 方法可以被客户端对象显式调用，也可以在容器**内部隐式**地被调用。

隐式调用有如下两种情况:

- **BeanFactory** 来说，对象实例化默认采用**延迟初始化**。通常情况下，当对象 A 被请求而需要第一次实例化的时候，如果它所依赖的对象 B 之前同样没有被实例化，那么容器会先实例化对象 A 所依赖的对象。这时容器内部就会首先实例化对象 B，以及对象 A 依赖的其他还没有实例化的对象。这种情况是容器内部调用**getBean**()，对于本次请求的请求方是隐式的。
- **ApplicationContext** 启动之后会**实例化所有单例 bean** 定义。但 ApplicationContext 在实现的过程中依然遵循 Spring 容器实现流程的两个阶段，只不过它会在启动阶段的活动完成之后，紧接着调用注册到该容器的所有 bean 定义的实例化方法 getBean()。这就是为什么当你得到 ApplicationContext 类型的容器引用时，容器内所有对象已经被**全部实例化**完成。 ApplicationContext 会利用反射机制自动识别出配置文件中定义的 BeanPostProcessor、InstantiationAwareBeanPostProcessor 和 BeanFactoryPostProcessor，并将它们自动注册到应用上下文中，而 BeanFactory 需要手工调用 addBeanPostProcessor() 方法进行注册。ApplicationContext 在启动时，将首先为配置文件中的每个 \<bean> 生成一个 **BeanDefinition** 对象，它是这个 Bean 在Spring 容器中的内部表示。

​       之所以说 getBean() 方法是有可能触发 Bean 实例化阶段的活动，是因为只有当对应某个  bean 定义的getBean() 方法第一次被调用时，不管是显式的还是隐式的，Bean实例化阶段的活动才会被触发，第二次被调用则会直接返回容器**缓存**的第一次实例化完的 **singleton** 对象实例（prototype 类型 bean 除外）。当 getBean() 方法内部发现该 bean 定义之前还没有被实例化之后，会通过 createBean() 方法来进行具体的对象实例化。



#### BeanFactory与FactoryBean

 两个特别像，但是功能却千差万别。有关于 **BeanFactory**，我们都知道，这是 Spring **容器的基础实现类**，它负责生产和管理 Bean 的一个**工厂**。当然 BeanFactory 只是一个**接口**，它的常用实现有 XmlBeanFactory、DefaultListableBeanFactory、**ApplicationContext **等。

![1573545313232](assets/1573545313232.png)

FactoryBean 是一个**接口**，具体方法如下： 

```java
public interface FactoryBean<T> {
 	// 返回由 FactoryBean 创建的Bean实例
	T getObject() throws Exception;
 	// 返回 FactoryBean 创建Bean的类型
	Class<?> getObjectType();
 	// 返回是否是 singleton
	boolean isSingleton();
}
```

**我们常规的 Bean 都是使用 Class 的反射获取具体实例，如果 Bean 的获取过程比较==复杂==，那么常规的 xml 配置需要配置大量属性值，这个时候我们就可以使用 FactoryBean，实现这个接口，在其 getObject() 方法中初始化这个 bean。** 

FactoryBean 使用实例：

```java
public class StudentFactoryBean implements FactoryBean {
 
    @Override
    public Object getObject() throws Exception {
        Student student = new Student();
        student.setAge(22);
        student.setName("jj");
        student.setId(10);
        return student;
    }
 
    // 对象具体类型
    @Override
    public Class<?> getObjectType() {
        return Student.class;
    }
 
    // 是否单例
    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

 在 SpringConfiguration 中添加该 bean 

```java
@Configuration
public class SpringConfiguration {
    // 添加Bean
    @Bean
    public StudentFactoryBean studentFactoryBean(){
        return new StudentFactoryBean();
    }
}
```

测试

```java
@Test
public void testStudentFactoryBean(){
    AnnotationConfigApplicationContext applicationContext
        = new AnnotationConfigApplicationContext(SpringConfiguration.class);
    Student student = (Student) applicationContext.getBean("studentFactoryBean");
    System.out.println(student);
}
// res
Student(id=10, name=test:jj, age=22)
```

小结：

- **BeanFactory：工厂类接口，Spring 容器的核心接口，实例化 bean、配置 bean 之间的依赖关系**

- **FactoryBean：实例化 bean 过程比较复杂时可以考虑使用**



#### Bean配置方式

 对于 Spring 来讲，为实现 Bean 的信息定义，提供了基于 XML、基于注解、基于 JAVA 类、基于 Groovy 这 4 种选项，同事还允许各种配置方式复合共存。 

几种配置方式对比：

![1573547101474](assets/1573547101474.png)



#### 容器内部工作机制

AbstractApplicationContext 是 ApplicationContext 的抽象实现类，该类的 refresh() 方法定义了 Spring 容器在加载配置文件后的各项处理过程，refresh() 方法的源码如下。

```java
public void refresh() throws BeansException, IllegalStateException {
        synchronized (this.startupShutdownMonitor) {
            // 做refresh之前的初始化工作，包括设定此context的状态等
            prepareRefresh();

            // 留给子类的两个模板方法，用于子类真正实现刷新和返回其内置BeanFactory
            ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

            // 设置BeanFactory的各个域值
            prepareBeanFactory(beanFactory);

            try {
                // Allows post-processing of the bean factory in context subclasses.
                postProcessBeanFactory(beanFactory);

                // 注册工厂后处理器
                invokeBeanFactoryPostProcessors(beanFactory);

                // 注册Bean后处理器
                registerBeanPostProcessors(beanFactory);

                // 初始化消息源
                initMessageSource();

                // 初始化应用上下文事件广播器
                initApplicationEventMulticaster();

                // 初始化其他特殊的bean
                onRefresh();

                // 注册事件监听器
                registerListeners();

                // 初始化所有单实例的bean
                finishBeanFactoryInitialization(beanFactory);

                // 完成刷新并发布容器刷新事件
                finishRefresh();
            }

            catch (BeansException ex) {
                if (logger.isWarnEnabled()) {
                    logger.warn("Exception encountered during context initialization - " +
                            "cancelling refresh attempt: " + ex);
                }

                // Destroy already created singletons to avoid dangling resources.
                destroyBeans();

                // Reset 'active' flag.
                cancelRefresh(ex);

                // Propagate exception to caller.
                throw ex;
            }

            finally {
                // Reset common introspection caches in Spring's core, since we
                // might not ever need metadata for singleton beans anymore...
                resetCommonCaches();
            }
        }
}
```



#### IOC容器

所有IOC容器都需要实现接口BeanFactory或其子接口，它是一个顶级容器接口。



##### BeanFactory源码分析

```java
package org.springframework.beans.factory;

/**
BeanFactory接口源码
*/
public interface BeanFactory {

    /**
     * 用来引用一个实例，或把它和工厂产生的Bean区分开，就是说，如果一个FactoryBean的名字为a，那么，&a会得到那个Factory
     */
    String FACTORY_BEAN_PREFIX = "&";       // 前缀

    /*
     * 四个不同形式的getBean方法，获取实例
     */
    Object getBean(String name) throws BeansException;

    <T> T getBean(String name, Class<T> requiredType) throws BeansException;

    <T> T getBean(Class<T> requiredType) throws BeansException;

    Object getBean(String name, Object... args) throws BeansException;

    boolean containsBean(String name);      // 是否包含Bean

    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;      // 是否为单实例

    boolean isPrototype(String name) throws NoSuchBeanDefinitionException;      // 是否为原型（多实例）

    boolean isTypeMatch(String name, Class<?> targetType) throws NoSuchBeanDefinitionException;   // 名称、类型是否匹配

    Class<?> getType(String name) throws NoSuchBeanDefinitionException;     // 获取类型

    String[] getAliases(String name);       // 根据实例的名字获取实例的别名

}
```

- bean默认单例存在。getBean方法返回的是同一个bean对象。
- 允许我们按名称或类型获取bean。
- ApplicationContext 是 BeanFactory 的子接口。
- AnnotationConfigApplicationContext 是一个基于注解的IOC容器。都包含上述的这些方法。



**Spring IoC容器的接口设计**

![1564994320924](assets/1564994320924.png)



#### Bean的装配

##### 基于Config的装配

定义一个普通的POJO类。

```java
/**
* 普通的User类 没有任何注解
*/
public class User {

	private Long id;
	private String userName;
    private String note;
    // Getter and setters
}
```

再自定义一个AppConfig类。

- **@Configuration** 标注指明类为配置类
- @Bean 将getDevDataSource()方法返回的POJO装配到IOC容器中 标注有 @Configuration 的类被认为为配置类，Spring自动根据其生成IoC容器来装配bean。

```java
/**
* 自定义的配置类
*/
@Configuration      // 标注指明类为配置类
public class AppConfig {
	// 指明Bean名称为user，否则使用方法名作为bean的名称
    @Bean(name = "user")
    public User initUser() {
        User user = new User();
        user.setId(1L);
        user.setUserName("Tom");
        user.setNote("Note 1");
        return user;
    }
}
```

以下通过AnnotationConfigApplicationContext容器来获取bean。此处AnnotationConfigApplicationContext是BeanFactory接口的子接口，也具有getBean方法来得到bean。此处比较适合第三方bean的装配，如上述的数据库连接bean的装配。

```java
public class IoCTest{
    public static void main(String[] args){
        // 传入配置类
        ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
        User user = ctx.getBean(User.class);
        System.out.println(user.getId());
    }
}
```

再看关于数据源的Bean装配，另外再定义AppConfig类。添加了不少注解。

- @Configuration 说明为一个配置类
- @ComponentScan 标明采用何种策略去**扫描装配** Bean，默认扫描AppConfig类当前所在的包及其子包。可以自定义扫描路径如下所示。配置项：basePackages 定义扫描的包名；basePackageClasses定义扫描的类；includeFilters定义满足条件的才扫描；excludeFIlters则是排除过滤器条件的bean，这两个需要@Filter注解去定义，它有一个type类型去定义条件。lazyInit 配置项配置是否延迟初始化。

```java
@Configuration
@ComponentScan(basePackages = "com.springboot.chapter3.*") // 此处的包名即为上述User类所在的包
@ImportResource(value = {"classpath:spring-other.xml"})
public class AppConfig {
	
	@Bean(name = "dataSource", destroyMethod = "close")
	@Profile("dev")
	public DataSource getDevDataSource() {
		Properties props = new Properties();
		props.setProperty("driver", "com.mysql.jdbc.Driver");
		props.setProperty("url", "jdbc:mysql://localhost:3306/dev_spring_boot");
		props.setProperty("username", "root");
		props.setProperty("password", "123456");
		DataSource dataSource = null;
		try {
			dataSource = BasicDataSourceFactory.createDataSource(props);
		} catch (Exception e) {
			e.printStackTrace();
		}
		return dataSource;
	}
}
```

其他的扫描路径方法示意。

```java
@ComponentScan("com.springboot.chapter3.*") 	// 此处的包名即为上述User类所在的包
@ComponentScan(basePackages = {"com.springboot.chapter3.*"}) 	// 此处的包名即为上述User类所在的包
@ComponentScan(basePackageClasses = {User.class}) 	// 根据类名扫描
@ComponentScan(basePackages = "com.springboot.chapter3.*", excludeFilters = {@Filter(classes = {User.class})}) 		// 自定义排除扫描条件 
```



##### 基于注解的装配

定义一个POJO类，这次的类带注解。直接在此类里面注入属性。注意与上面的User类对比。

- @Component 标明哪个类被扫描到容器中
- @Value 字段中注入属性值

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

/**
* User类
*/
@Component("user")      // 指定user为bean名称 如果不指定则把类名首字母小写当做类名称
public class User {

	@Value("1")         // @Value注入属性值
	private Long id;
	@Value("user_name_1")
	private String userName;
	@Value("note_1")
	private String note;

	// Getters and Setters
}
```



#### 依赖注入

##### 注解@Autowired

getBean() 方法支持根据类型和名称来获取对应的 bean。@Autowired 注解首先根据类型去**寻找对应的 bean**，找不到再根据**属性名称和 bean 名称**来寻找 bean。默认必须找到对应 Bean，否则报错（可以使用 required = false 关闭必须装配）。

可以标注在**属性**上，也可以标注在方法上，还可以标注在入参上。

```java
@Autowired	
private Animal animal = null;
```

```java
@Override
@Autowired
public void setAnimal(Animal animal) {
    this.animal = animal;
}
```

```java
public BussinessPerson(@Autowired Animal animal) {
    this.animal = animal;
}
```

##### 使用@Primary与@Qualifier消除歧义问题

两个接口：动物与人接口。

```java
public interface Person {
	
	public void service();
	
	public void setAnimal(Animal animal);
	
}
```

```java
public interface Animal {
	public void use();
}
```

实现接口

**狗类**

```java
@Component
public class Dog implements Animal {

	@Override
	public void use() {
		System.out.println("狗【" + Dog.class.getSimpleName()+"】是看门用的。");
	}
}
```

```java
@Component
public class BussinessPerson implements Person{
    // 自动注入实现了动物接口的类
    @Autowired	
    private Animal animal = null;

    @Override
    public void service() {
        this.animal.use();
    }

    @Override
    public void setAnimal(Animal animal) {
        this.animal = animal;
    }
}
```

上述 BussinessPerson 中自动注入实现了动物接口的类，此时容器中实现了 Animal 接口的**只有 Dog 类**，因此注入 Dog 的实例。

如果再实现一个动物类。

**猫类**

```java
@Component
public class Cat implements Animal {

	@Override
	public void use() {
		System.out.println("猫【" + Cat.class.getSimpleName()+"】是抓老鼠。");
	}
}
```

此时同时有 Dog 类和 Cat 类实现了 Animal 接口。

此时 BussinessPerson 的自动注入会报错，因为**不知道注入哪一个**实例。产生注入失败是因为**按类型查找**，动物 Animal 接口有多个类型，这就是存在歧义。

###### @Primary 与 @Qualifier 注解

注解 @Primary 可以修改**优先权**。比如在 Cat 类上使用此注解。

```java
@Component
@Primary
public class Cat implements Animal {...}
```

此时容器会优先注入 Cat 实例到 Animal 中。

@Primary 也可以用在多个类上，此时也会有歧义，可以使用 **@Qualifier 注解**。

@Qualifier 注解的配置项 **value** 需要一个字符串去定义，它可以与 @Autowired 一起去通过**类型域名称一起寻找 Bean**。如下。此时注入的就是 Dog 类的实例。

```java
@Autowired
@Qualifier("dog")
Animal animal = null;
```



#### 生命周期

- Bean定义、Bean初始化、Bean生存期、Bean销毁。

##### 1. Spring初始化Bean流程

- 资源定位(例如 @ComponentScan 所定义的扫描包)
- Bean定义(将 Bean 的定义保存到 BeanDefinition 的实例中)
- 发布 Bean 定义( IOC 容器装载 Bean 的定义)
- 实例化(创建 Bean 的实例对象)
- 依赖注入(例如 @Autowired 类的各类资源)

![1564994604731](assets/1564994604731.png)

##### 2. Spring Bean的生命周期

![1564994754829](assets/1564994754829.png)

- Bean容器找到配置文件中 Spring Bean 的定义。
- Bean容器利用 Java Reflection API 创建一个 Bean 的实例。
- 如果涉及到一些属性值 利用 set 方法设置一些属性值。
- 如果Bean实现了 BeanNameAware 接口，调用 setBeanName() 方法，传入 Bean 的名字。
- 如果 Bean 实现了 BeanClassLoaderAware 接口，调用 setBeanClassLoader() 方法，传入 ClassLoader 对象的实例。
- 如果 Bean 实现了 BeanFactoryAware 接口，调用 setBeanFactory() 方法，传入 BeanFactory 对象的实例。
- 与上面的类似，如果实现了**其他 *Aware 接口**，就调用相应的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 **BeanPostProcessor** 对象，执行 postProcessBeforeInitialization()方法。
- 如果 Bean 实现了 **InitializingBean** 接口，执行 afterPropertiesSet() 方法。
- 如果 Bean 在配置文件中的定义包含 init-method 属性，执行指定的方法。
- 如果有和加载这个 Bean 的 Spring 容器相关的 **BeanPostProcessor** 对象，执行 postProcessAfterInitialization() 方法
- 当要销毁 Bean 的时候，如果 Bean 实现了 **DisposableBean** 接口，执行 destroy() 方法。
- 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。

![image-20200528144152364](assets/image-20200528144152364.png)

<img src="assets/image-20200528144205341.png" alt="image-20200528144205341" style="zoom:90%;" />

若容器注册了以上各种接口，程序那么将会按照以上的流程进行。下面将仔细讲解各接口作用。

##### 3. 各个接口方法分类

Bean 的完整生命周期经历了各种方法调用，这些方法可以划分为以下几类：

- Bean自身的方法：这个包括了 Bean 本身调用的方法和通过配置文件中 **\<bean>** 的 **init-method** 和 **destroy-method** 指定的方法。

- Bean 级生命周期接口方法：这个包括了 BeanNameAware、BeanFactoryAware、InitializingBean 和DiposableBean 这些接口的方法。

- 容器级生命周期接口方法：这个包括了 **InstantiationAwareBeanPostProcessor** 和 **BeanPostProcessor** 这两个接口实现，一般称它们的实现类为“**后处理器**”。

- 工厂后处理器接口方法：这个包括了 AspectJWeavingEnabler, ConfigurationClassPostProcessor, CustomAutowireConfigurer 等等非常有用的**工厂后处理器接口**的方法。工厂后处理器也是**容器级**的。在应用上下文装配配置文件之后立即调用。

##### 4. initialization和destroy

有时我们需要在 Bean 属性值 set 好之后和 Bean 销毁之前做一些事情，比如检查 Bean 中某个属性是否被正常的设置好值了。Spring 框架提供了多种方法让我们可以在 Spring Bean 的生命周期中执行 initialization 和 pre-destroy 方法。

**1. 实现InitializingBean和DisposableBean接口**

这两个接口都只包含**一个方法**。通过实现 **InitializingBean** 接口的 afterPropertiesSet() 方法可以在 Bean 属性值设置好之后做一些操作，实现 DisposableBean 接口的 destroy() 方法可以在销毁 Bean 之前做一些操作。

例子如下：

```java
public class GiraffeService implements InitializingBean,DisposableBean {
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("执行InitializingBean接口的afterPropertiesSet方法");
    }
    @Override
    public void destroy() throws Exception {
        System.out.println("执行DisposableBean接口的destroy方法");
    }
}
```

这种方法比较简单，但是**不建议使用**。因为这样会将 Bean 的实现和 Spring 框架**耦合在一起**。

**2. 在bean的配置文件中指定init-method和destroy-method方法**

Spring允许我们创建自己的 init 方法和 destroy 方法，只要在 Bean 的配置文件中指定 init-method 和 destroy-method 的值就可以在 Bean 初始化时和销毁之前执行一些操作。

例子如下：

```java
public class GiraffeService {
    //通过<bean>的destroy-method属性指定的销毁方法
    public void destroyMethod() throws Exception {
        System.out.println("执行配置的destroy-method");
    }
    //通过<bean>的init-method属性指定的初始化方法
    public void initMethod() throws Exception {
        System.out.println("执行配置的init-method");
    }
}
```

配置文件中的配置：

```
<bean name="giraffeService" class="com.giraffe.spring.service.GiraffeService" init-method="initMethod" destroy-method="destroyMethod">
</bean>
```

需要注意的是自定义的init-method和post-method方法可以抛异常但是不能有参数。

这种方式比较推荐，因为可以自己创建方法，无需将Bean的实现直接依赖于spring的框架。

**3. 使用@PostConstruct和@PreDestroy注解**

除了xml配置的方式，Spring 也支持用 `@PostConstruct`和 `@PreDestroy`注解来指定 `init` 和 `destroy` 方法。这两个注解均在`javax.annotation` 包中。为了注解可以生效，需要在配置文件中定义org.springframework.context.annotation.**CommonAnnotationBeanPostProcessor** 或 context:annotation-config

例子如下：

```java
public class GiraffeService {
    @PostConstruct
    public void initPostConstruct(){
        System.out.println("执行PostConstruct注解标注的方法");
    }
    @PreDestroy
    public void preDestroy(){
        System.out.println("执行preDestroy注解标注的方法");
    }
}
```

配置文件：

```xml
<bean class="org.springframework.context.annotation.CommonAnnotationBeanPostProcessor" />
```

##### 5. 实现*Aware接口在Bean中使用Spring框架的一些对象

有些时候我们需要在 **Bean 的初始化**中使用 **Spring ==框架自身==的一些对象**来执行一些操作，比如获取 ServletContext 的一些参数，获取 ApplicaitionContext 中的 BeanDefinition 的名字，获取 Bean 在容器中的名字等等。**==为了让 Bean 可以获取到框架自身的一些对象，Spring 提供了一组名为 *Aware 的接口。==**

这些接口均继承于 `org.springframework.beans.factory.Aware ` 标记接口，并提供一个将**由 Bean 实现的 set* 方法，** Spring 通过基于 setter 的**依赖注入方式使相应的对象可以被 Bean 使用**。

介绍一些**重要的 Aware 接口**：

- **ApplicationContextAware**：获得 **ApplicationContext** 对象，可以用来获取所有 BeanDefinition 的名字。
- **BeanFactoryAware**：获得 **BeanFactory** 对象，可以用来检测 Bean 的作用域。
- **BeanNameAware**：获得 Bean 在**配置文件**中定义的名字。
- **ResourceLoaderAware**：获得 ResourceLoader 对象，可以获得 **classpath** 中某个文件。
- **ServletContextAware**：在一个 MVC 应用中可以获取 **ServletContext** 对象，可以读取 context 中的参数。
- **ServletConfigAware**：在一个 MVC 应用中可以获取 **ServletConfig** 对象，可以读取 config 中的参数。

以下是实现上述接口的例子。

```java
public class GiraffeService implements ApplicationContextAware,
ApplicationEventPublisherAware, BeanClassLoaderAware, BeanFactoryAware,
BeanNameAware, EnvironmentAware, ImportAware, ResourceLoaderAware{
    @Override
    public void setBeanClassLoader(ClassLoader classLoader) {
        System.out.println("执行setBeanClassLoader,ClassLoader Name = " + classLoader.getClass().getName());
    }
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("执行setBeanFactory,setBeanFactory:: giraffe bean singleton=" +  beanFactory.isSingleton("giraffeService"));
    }
    @Override
    public void setBeanName(String s) {
        System.out.println("执行setBeanName:: Bean Name defined in context=" + s);
    }
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("执行setApplicationContext:: Bean Definition Names="
         + Arrays.toString(applicationContext.getBeanDefinitionNames()));
    }
    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        System.out.println("执行setApplicationEventPublisher");
    }
    @Override
    public void setEnvironment(Environment environment) {
        System.out.println("执行setEnvironment");
    }
    @Override
    public void setResourceLoader(ResourceLoader resourceLoader) {
        Resource resource = resourceLoader.getResource("classpath:spring-beans.xml");
        System.out.println("执行setResourceLoader:: Resource File Name="
                           + resource.getFilename());
    }
    @Override
    public void setImportMetadata(AnnotationMetadata annotationMetadata) {
        System.out.println("执行setImportMetadata");
    }
}
```

##### 6. BeanPostProcessor

上面的 ***Aware** 接口是针对**==某个==实现这些接口的 Bean 定制初始化**的过程，Spring 同样可以针对容器中的所有 Bean，或者**==某些== Bean** 定制初始化过程，只需**提供一个实现 BeanPostProcessor 接口的类**即可。 该接口中包含两个方法， postProcessBeforeInitialization 和 postProcessAfterInitialization。 

```java
public interface BeanPostProcessor {
    @Nullable
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Nullable
    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
```

postProcessBeforeInitialization 方法会在容器中的 **Bean 初始化之前执行，** postProcessAfterInitialization 方法在容器中的 **Bean 初始化之后**执行。

例子如下：

```java
public class CustomerBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("执行BeanPostProcessor的postProcessBeforeInitialization方法,beanName=" + beanName);
        return bean;
    }
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("执行BeanPostProcessor的postProcessAfterInitialization方法,beanName=" + beanName);
        return bean;
    }
}
```

要将 BeanPostProcessor 的 Bean 像其他 Bean 一样定义在配置文件中

```xml  
<bean class="com.giraffe.spring.service.CustomerBeanPostProcessor"/>
```

##### 7. 单例与非单例对象管理

**其实很多时候我们并不会真的去实现上面说描述的那些接口，那么下面我们就除去那些接口，针对bean的单例和非单例来描述下bean的生命周期：**

###### ① 单例管理的对象

当scope=”singleton”，即默认情况下，会在启动容器时（即实例化容器时）时实例化。但我们可以指定Bean节点的lazy-init=”true”来延迟初始化bean，这时候，只有在第一次获取bean时才会初始化bean，即第一次请求该bean时才初始化。如下配置：

```xml
<bean id="ServiceImpl" class="cn.csdn.service.ServiceImpl" lazy-init="true"/>  
```

如果想对所有的默认单例bean都应用延迟初始化，可以在根节点beans设置default-lazy-init属性为true，如下所示：

```xml
<beans default-lazy-init="true" …>
```

默认情况下，Spring 在读取 xml 文件的时候，就会创建对象。在创建对象的时候先调用构造器，然后调用 init-method 属性值中所指定的方法。对象在被销毁的时候，会调用 destroy-method 属性值中所指定的方法（例如调用Container.destroy()方法的时候）。写一个测试类，代码如下：

```java
public class LifeBean {
    private String name;  

    public LifeBean(){  
        System.out.println("LifeBean()构造函数");  
    }  
    public String getName() {  
        return name;  
    }  

    public void setName(String name) {  
        System.out.println("setName()");  
        this.name = name;  
    }  

    public void init(){  
        System.out.println("this is init of lifeBean");  
    }  

    public void destory(){  
        System.out.println("this is destory of lifeBean " + this);  
    }  
}
```

　life.xml配置如下：

```xml
<bean id="life_singleton" class="com.bean.LifeBean" scope="singleton" 
            init-method="init" destroy-method="destory" lazy-init="true"/>
```

测试代码：

```java
public class LifeTest {
    @Test 
    public void test() {
        AbstractApplicationContext container = 
        new ClassPathXmlApplicationContext("life.xml");
        LifeBean life1 = (LifeBean)container.getBean("life");
        System.out.println(life1);
        container.close();
    }
}
```

运行结果：

```
LifeBean()构造函数
this is init of lifeBean
com.bean.LifeBean@573f2bb1
……
this is destory of lifeBean com.bean.LifeBean@573f2bb1
```

###### ② 非单例管理的对象

当`scope=”prototype”`时，容器也会延迟初始化 bean，Spring 读取xml 文件的时候，并不会立刻创建对象，而是在第一次请求该 bean 时才初始化（如调用getBean方法时）。在第一次请求每一个 prototype 的bean 时，Spring容器都会调用其构造器创建这个对象，然后调用`init-method`属性值中所指定的方法。对象销毁的时候，Spring 容器不会帮我们调用任何方法，因为是非单例，这个类型的对象有很多个，Spring容器一旦把这个对象交给你之后，就不再管理这个对象了。

为了测试prototype bean的生命周期life.xml配置如下：

```xml
<bean id="life_prototype" class="com.bean.LifeBean" scope="prototype" init-method="init" destroy-method="destory"/>
```

测试程序：

```java
public class LifeTest {
    @Test 
    public void test() {
        AbstractApplicationContext container = new ClassPathXmlApplicationContext("life.xml");
        LifeBean life1 = (LifeBean)container.getBean("life_singleton");
        System.out.println(life1);

        LifeBean life3 = (LifeBean)container.getBean("life_prototype");
        System.out.println(life3);
        container.close();
    }
}
```

运行结果：

```
LifeBean()构造函数
this is init of lifeBean
com.bean.LifeBean@573f2bb1
LifeBean()构造函数
this is init of lifeBean
com.bean.LifeBean@5ae9a829
……
this is destory of lifeBean com.bean.LifeBean@573f2bb1
```

可以发现，对于作用域为 prototype 的 bean ，其`destroy`方法并没有被调用。**如果 bean 的 scope 设为prototype时，当容器关闭时，`destroy` 方法不会被调用。对于 prototype 作用域的 bean，有一点非常重要，那就是 Spring不能对一个 prototype bean 的整个生命周期负责：容器在初始化、配置、装饰或者是装配完一个prototype实例后，将它交给客户端，随后就对该prototype实例不闻不问了。** 不管何种作用域，容器都会调用所有对象的初始化生命周期回调方法。但对prototype而言，任何配置好的析构生命周期回调方法都将不会被调用。**清除prototype作用域的对象并释放任何prototype bean所持有的昂贵资源，都是客户端代码的职责**（让Spring容器释放被prototype作用域bean占用资源的一种可行方式是，通过使用bean的后置处理器，该处理器持有要被清除的bean的引用）。谈及prototype作用域的bean时，在某些方面你可以将Spring容器的角色看作是Java new操作的替代者，任何迟于该时间点的生命周期事宜都得交由客户端来处理。

**Spring 容器可以管理 singleton 作用域下 bean 的生命周期，在此作用域下，Spring 能够精确地知道bean何时被创建，何时初始化完成，以及何时被销毁。而对于 prototype 作用域的bean，Spring只负责创建，当容器创建了 bean 的实例后，bean 的实例就交给了客户端的代码管理，Spring容器将不再跟踪其生命周期，并且不会管理那些被配置成prototype作用域的bean的生命周期。**

##### 8. demo

参考这个：https://www.cnblogs.com/zrtqsk/p/3735273.html



#### 使用属性文件

##### application.properties

是默认的配置文件，yml 文件也是类似的效果。

```properties
# application.properties中
database.url = jdbc:mysql://localhost:3306/test_db
database.username = root
database.password = 123456
```

可以在类中注入属性文件中的值

```java
@Component
public class DatabaseProperties{
    
    @Value("${database.url}")   // 此处引用配置文件中的属性并注入
    private String url = null;
}
```

如果都用上述的方法可能需要写很多次，因此可以用如下方法直接指定属性文件中的值，如database，然后用全限定名去定位值，如database.url。

```java
@Component
@ConfigurationProperties("database")    // 使用此注解传入字符串database，会去配置文件寻找对应的属性自动注入
public class DatabaseProperties{
    
    private String url = null;
    
    // Getters and Setters
}
```

##### 自定义属性文件

如把数据库连接信息放入 jdbc.properties 中，然后使用 @PropertySource 去定义对应的属性文件, @PropertySource 注解需要在配置类中标注，value 值可以有多个。ignoreResourceNotFound 指示找不到文件就忽略。

```java
@SpringBootApplication
@ComponentScan(basePackages = {"com.springboot.chapter"})
@PropertySource(value = {"classpath:jdbc.properties"}, ignoreResourceNotFound = ture)  // 寻找配置文件
public class IoCTest{
    public static void main(String[] args){
        SpringApplication.run(IoCTest.class, args);
    }
}
```



#### 条件装配Bean

满足一定条件才装配 Bean，否则不装配，比如数据库配置信息不全就不装配。

##### @Conditional 注解

用于条件装配 Bean，需要配合 Condition 接口使用。

```java
@Configuration
@ComponentScan(basePackages = "com.springboot.chapter3.*")
@ImportResource(value = {"classpath:spring-other.xml"})
public class AppConfig {
	
	@Bean(name = "dataSource", destroyMethod = "close")
	@Conditional(DatabaseConditional.class)     // 传入的类需实现Condition接口 
	public DataSource getDataSource(            // 使用@Value去取配置文件中的值并注入
			@Value("${database.driverName}") String driver,
			@Value("${database.url}") String url,
			@Value("${database.username}") String username, 
			@Value("${database.password}") String password
			) {
		Properties props = new Properties();
		props.setProperty("driver", driver);
		props.setProperty("url", url);
		props.setProperty("username", username);
		props.setProperty("password", password);
		DataSource dataSource = null;
		try {
			dataSource = BasicDataSourceFactory.createDataSource(props);
		} catch (Exception e) {
			e.printStackTrace();
		}
		return dataSource;
	}
}	
```

上述的

> @Conditional(DatabaseConditional.class)

传入了DatabaseConditional 类，传入 @Conditional 注解的类需要实现 Condition 接口。

```java
import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.env.Environment;
import org.springframework.core.type.AnnotatedTypeMetadata;

public class DatabaseConditional implements Condition {

    /**
	 * 数据库装配条件 不符合条件的就不装配
	 * @param context 条件上下文
	 * @param 
	 */
	@Override
	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
		Environment env = context.getEnvironment();
        // 此处是判断条件
		return env.containsProperty("database.driverName") && env.containsProperty("database.url") 
				&& env.containsProperty("database.username") && env.containsProperty("database.password");
	}
}
```

只有满足了**上述的条件**，才会**装配** DataSource 。



#### Bean的作用域

**Spring 中的 bean 默认都是单例的，这些单例 Bean 在多线程程序下如何保证线程安全呢？** 例如对于 Web 应用来说， Web 容器对于每个用户请求都创建一个**单独的 Sevlet 线程**来处理请求，引入 Spring 框架之后，每个 Action 都是**单例**的，那么对于 Spring 托管的单例 Service Bean，如何保证其安全呢？ **Spring 的单例是基于 BeanFactory 也就是 Spring 容器的，单例 Bean 在此容器内==只有一个==，Java 的单例是基于 JVM，每个 JVM 内只有一个实例。**

在大多数情况下。单例 bean 是很理想的方案。不过，有时候你可能会发现你所使用的类是易变的，它们**会保持一些状态**，因此重用是不安全的。在这种情况下，将 class 声明为单例的就不是那么明智了。因为对象会被污染，稍后重用的时候会出现意想不到的问题。所以 Spring 定义了多种作用域的bean。

##### 1. 作用域分类

|     Scope      |                         Description                          |
| :------------: | :----------------------------------------------------------: |
| **singleton**  | （默认的）在每个Spring IoC容器中，一个bean定义对应只会有唯一的一个bean实例 |
| **prototype**  |                一个bean定义可以有多个bean实例                |
|  **request**   | 一个 bean 定义对应于单个 HTTP 请求的生命周期，仅适用于WebApplicationContext环境。 |
|  **session**   |      一个 bean 定义对应于单个 HTTP Session 的生命周期。      |
| global session |           仅仅在基于 portlet 的 web 应用中才有意义           |

五种作用域中，**request、session** 和 **global session** 三种作用域仅在基于 web 的应用中使用（不必关心你所采用的是什么 web 应用框架），只能用在基于 web 的 Spring ApplicationContext 环境。

##### 2. singleton

唯一 bean 实例。

**当一个 bean 的作用域为 singleton，那么Spring IoC容器中只会存在一个共享的 bean 实例，并且所有对 bean 的请求，只要 id 与该 bean 定义相匹配，则只会返回bean的同一实例。** singleton 是单例类型(对应于单例模式)，就是在创建起容器时就同时自动创建了一个bean的对象，不管你是否使用，但我们可以指定Bean节点的 `lazy-init=”true”` 来延迟初始化bean，这时候，只有在第一次获取bean时才会初始化bean，即第一次请求该bean时才初始化。 每次获取到的对象都是同一个对象。注意，singleton 作用域是Spring中的缺省作用域。要在XML中将 bean 定义成 singleton ，可以这样配置：

```xml
<bean id="ServiceImpl" class="cn.csdn.service.ServiceImpl" scope="singleton">
```

也可以通过 `@Scope` 注解（它可以显示指定bean的作用范围。）的方式

```java
@Service
@Scope("singleton")
public class ServiceImpl{

}
```

##### 3. prototype

每次请求都会创建一个新的 bean 实例。

**当一个bean的作用域为 prototype，表示一个 bean 定义对应多个对象实例。** **prototype 作用域的 bean 会导致在每次对该 bean 请求**（将其注入到另一个 bean 中，或者以程序的方式调用容器的 getBean() 方法**）时都会创建一个新的 bean 实例。prototype 是原型类型，它在我们创建容器的时候并没有实例化，而是当我们获取bean的时候才会去创建一个对象，而且我们每次获取到的对象都不是同一个对象。根据经验，对有状态的 bean 应该使用 prototype 作用域，而对无状态的 bean 则应该使用 singleton 作用域。**  在 XML 中将 bean 定义成 prototype ，可以这样配置：

```java
<bean id="account" class="com.foo.DefaultAccount" scope="prototype"/>  
 或者
<bean id="account" class="com.foo.DefaultAccount" singleton="false"/> 
```

通过 `@Scope` 注解的方式实现就不做演示了。

##### 4. request

每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP request 内有效。

**request只适用于Web程序，每一次 HTTP 请求都会产生一个新的bean，同时该bean仅在当前HTTP request内有效，当请求结束后，该对象的生命周期即告结束。** 在 XML 中将 bean 定义成 request ，可以这样配置：

```java
<bean id="loginAction" class=cn.csdn.LoginAction" scope="request"/>
```

##### 5. session

每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP session 内有效。

**session只适用于Web程序，session 作用域表示该针对每一次 HTTP 请求都会产生一个新的 bean，同时该 bean 仅在当前 HTTP session 内有效.与request作用域一样，可以根据需要放心的更改所创建实例的内部状态，而别的 HTTP session 中根据 userPreferences 创建的实例，将不会看到这些特定于某个 HTTP session 的状态变化。当HTTP session最终被废弃的时候，在该HTTP session作用域内的bean也会被废弃掉。**

```xml
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>
```

##### 6. globalSession

global session 作用域类似于标准的 HTTP session 作用域，不过仅仅在基于 portlet 的 web 应用中才有意义。Portlet 规范定义了全局 Session 的概念，它被所有构成某个 portlet web 应用的各种不同的 portle t所共享。在global session 作用域中定义的 bean 被限定于全局portlet Session的生命周期范围内。

```xml
<bean id="user" class="com.foo.Preferences "scope="globalSession"/>
```

##### 7. @Scope 注解

如下所示为一个POJO类使用 @Scope 注解。使用 ConfigurableBeanFactory 只能提供单例 (SCOPE_SINGLETON)和原型 (SCOPE_PROTOTYPE)。在MVC中，还可以使用 WebApplicationContext 去提供MVC下特有的作用域形式，如请求（SCOPE_REQUEST），会话（SCOPE_SESSION）。

```java
import org.springframework.beans.factory.config.ConfigurableBeanFactory;
import org.springframework.context.annotation.Profile;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

@Component
@Scope(ConfigurableBeanFactory.SCOPE_SINGLETON)  	// 设置bean类型
// @Scope(WebApplicationContext.SCOPE_REQUSET)  	// 设置bean类型(For MVC)
public class ScopeBean {
}
```



#### 环境切换

##### @Profile注解

使用Profile机制，在各个环境之间转换。使用 @Profile 注解可以实现不同环境下配置参数的切换，如生产环境与开发环境。任何@Component或@Configuration注解的类都可以使用@Profile注解。 如

```java
@Configuration
@ComponentScan(basePackages = "com.springboot.chapter3.*")
@ImportResource(value = {"classpath:spring-other.xml"})
public class AppConfig {
	
	// 生产环境中使用的数据库连接
	@Bean(name = "dataSource", destroyMethod = "close")
	@Profile("dev")
	public DataSource getDevDataSource() {
		Properties props = new Properties();
		props.setProperty("driver", "com.mysql.jdbc.Driver");
		props.setProperty("url", "jdbc:mysql://localhost:3306/dev_spring_boot");
		props.setProperty("username", "root");
		props.setProperty("password", "123456");
		DataSource dataSource = null;
		try {
			dataSource = BasicDataSourceFactory.createDataSource(props);
		} catch (Exception e) {
			e.printStackTrace();
		}
		return dataSource;
	}
	
	// 测试环境中使用的数据库连接
	@Bean(name = "dataSource", destroyMethod = "close")
	@Profile("test")
	public DataSource getTestDataSource() {
		Properties props = new Properties();
		props.setProperty("driver", "com.mysql.jdbc.Driver");
		props.setProperty("url", "jdbc:mysql://localhost:3306/test_spring_boot");
		props.setProperty("username", "root");
		props.setProperty("password", "123456");
		DataSource dataSource = null;
		try {
			dataSource = BasicDataSourceFactory.createDataSource(props);
		} catch (Exception e) {
			e.printStackTrace();
		}
		return dataSource;
	}
}
```

在Java启动项目中，可以使用如下配置开启Profile机制。

```
JAVA_OPTS="-Dspring.profiles.active=dev"
```



#### 使用Spring EL

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
 
@Component("customerBean")
public class Customer {
 
    @Value("#{'lei'.toUpperCase()}")        	// Spring EL
    private String name;
 
    @Value("#{priceBean.getSpecialPrice()}")    // Spring EL
    private double amount;
    
    @Value("${database.driverName}")        	// Spring EL
    String driver = null;
    
    // getter and setter...省略
 
    @Override
    public String toString() {
        return "Customer [name=" + name + ", amount=" + amount + "]";
    }
 
}
```

- @Value中的 ${...} 代表占位符，它会读取上下文的属性值装配到属性中, 如属性文件中的值。
- \#{...}  代表启用Spring表达式，它将具有运算的功能。



#### 添加组件的注解

往 IOC 容器**添加组件**的注解。

① 通过 @CompentScan +@Controller @Service @Respository @compent

适用场景: 针对我们**自己写的组件**可以通过该方式来进行加载到容器中。

② 通过 @Bean 的方式来导入组件(适用于导入**第三方组件**的类)。

③ 通过 @Import 来**导入组件** （导入组件的 id 为全类名路径）。也可以导入**第三方**组件。

```java
@Configuration
@Import(value = {Person.class, Car.class})
public class MainConfig {
}
```

通过 @Import 的 ImportSeletor 类实现组件的导入 (导入组件的 id 为全类名路径)  。**自动装配**原理经常使用。

```java
public class TulingImportSelector implements ImportSelector {
    // 可以获取导入类的注解信息
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{"com.tuling.testimport.compent.Dog"};
    }
}   
```

使用这个 TulingImportSelector。

```java
@Configuration
@Import(value = {Person.class, Car.class, TulingImportSelector.class})
public class MainConfig {
}
```

通过 @Import 的 **ImportBeanDefinitionRegister** 导入组件 (可以指定 bean 的名称）。Bean **定义注册器**。

```java
public class TulingBeanDefinitionRegister implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        // 创建一个bean定义对象
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(Cat.class);
        // 把bean定义对象导入到容器中
        registry.registerBeanDefinition("cat", rootBeanDefinition);
    }
}
```

```java
@Configuration
//@Import(value = {Person.class, Car.class})
//@Import(value = {Person.class, Car.class, TulingImportSelector.class})
@Import(value = {Person.class, Car.class, TulingImportSelector.class, TulingBeanDefinitionRegister.class})
public class MainConfig {
}
```

④ 通过实现 **FacotryBean 接口**来实现添加组件。整合第三方的复杂初始化对象。典型的是 SqlSessionFactoryBean 组件。

```java
public class CarFactoryBean implements FactoryBean<Car> {
    // 返回bean的对象
    @Override
    public Car getObject() throws Exception {
        return new Car();
    } 
    // 返回bean的类型
    @Override
    public Class<?> getObjectType() {
        return Car.class;
    } 
    // 是否为单例
    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

注入这个组件。

```java
@Configuration
@ImportResource(locations = {"classpath:beans.xml"})
public class MainConfig {

    @Bean
    public CarFactoryBean carFactoryBean() {
        return new CarFactoryBean();
    }
}
```



#### 属性值的设置

通过 @Value +@PropertySource 来给组件赋值。

先来个 properties 文件。

```java
person.lastName=Jack
```

```java
public class Person {
    // 通过普通的方式
    @Value("Tom")
    private String firstName;
    // spel方式来赋值
    @Value("#{28-8}")
    private Integer age;
    // 通过读取外部配置文件的值
    @Value("${person.lastName}")
    private String lastName;
}
```

这里需要把配置文件加载到容器中。

```java
@Configuration
@PropertySource(value = {"classpath:person.properties"}) // 指定外部配置文件的位置
public class MainConfig {
    @Bean
    public Person person() {
        return new Person();
    }
}
```



#### 面试题

##### 1. 谈谈对IOC的理解？

IoC（Inverse of Control:控制反转）是一种**设计思想**，就是 **将原本在程序中手动创建对象的控制权，交由Spring框架来管理。**  IoC 在其他语言中也有应用，并非 Spring 特有。 **IoC 容器是 Spring 用来实现 IoC 的载体，  IoC 容器实际上就是个Map（key，value）,Map 中存放的是各种对象。**

IoC （Inversion of control ）控制反转/反转控制。它是一种思想不是一个技术实现。描述的是：Java 开发领域对象的创建以及管理的问题。

例如：现有类 A 依赖于类 B

- **传统的开发方式** ：往往是在类 A 中手动通过 new 关键字来 new 一个 B 的对象出来
- **使用 IoC 思想的开发方式** ：不通过 new 关键字来创建对象，而是通过 IoC 容器(Spring 框架) 来帮助我们实例化对象。我们需要哪个对象，直接从 IoC 容器里面过去即可。

从以上两种开发方式的对比来看：我们 “丧失了一个权力” (创建、管理对象的权力)，从而也得到了一个好处（不用再考虑对象的创建、管理等一系列的事情）

> 为什么叫控制反转？

- **控制** ：指的是对象创建（实例化、管理）的权力

- **反转** ：控制权交给外部环境（Spring 框架、IoC 容器）

将对象之间的相互依赖关系交给 IoC 容器来管理，并由 IoC 容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。  **IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。** 在实际项目中一个 Service 类可能有几百甚至上千个类作为它的底层，假如我们需要实例化这个 Service，你可能要每次都要搞清这个 Service 所有底层类的构造函数，这可能会把人逼疯。如果利用 IoC 的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。

Spring 时代我们一般通过 XML 文件来配置 Bean，后来开发人员觉得 XML 文件来配置不太好，于是 SpringBoot 注解配置就慢慢开始流行起来。

推荐阅读：https://www.zhihu.com/question/23277575/answer/169698662

**Spring IoC的初始化过程：** 

<img src="assets/image-20200528120916014.png" alt="image-20200528120916014" style="zoom:80%;" />

> IOC带来了什么好处？

IoC 的思想就是两方之间不互相依赖，由第三方容器来管理相关资源。这样有什么好处呢？

1. 对象之间的耦合度或者说依赖程度降低；
2. 资源变的容易管理；比如你用 Spring 容器提供的话很容易就可以实现一个单例。

使用 IoC 的思想，我们将对象的控制权（创建、管理）交有 IoC 容器去管理，我们在使用的时候直接向 IoC 容器 “要” 就可以了。

> IOC与DI的关系？

IoC（Inverse of Control:控制反转）是一种**设计思想** 或者说是某种模式。这个设计思想就是 **将原本在程序中手动创建对象的控制权，交由 Spring 框架来管理。** IoC 在其他语言中也有应用，并非 Spring 特有。**IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个 Map（key，value）,Map 中存放的是各种对象。**

**IoC 最常见以及最合理的实现方式叫做依赖注入（Dependency Injection，简称 DI）。**DI(Dependecy Inject,依赖注入)是实现控制反转的一种设计模式，依赖注入就是将实例变量传入到一个对象中去。

##### 2. Spring中的单例bean的线程安全问题了解吗？

大部分时候我们并没有在系统中使用多线程，所以很少有人会关注这个问题。单例 bean 存在线程问题，主要是因为当多个线程操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。

常见的有两种解决办法：

1. 在 Bean 对象中尽量避免定义可变的成员变量（不太现实）。

2. 在类中定义一个 **ThreadLocal** 成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的一种方式）。

##### 3. @Component和@Bean的区别是什么？

1. 作用对象不同: `@Component` 注解作用于类，而 `@Bean` 注解作用于方法。
2. `@Component` 通常是通过类路径扫描来自动侦测以及自动装配到 Spring 容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。`@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean,`@Bean`告诉了 Spring 这是某个类的示例，当我需要用它的时候还给我。
3. `@Bean` 注解比 `Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册 bean。比如当我们引用第三方库中的类需要装配到 `Spring` 容器时，则只能通过 `@Bean`来实现。

##### 4. 将一个类声明为Spring的bean的注解有哪些?

我们一般使用 `@Autowired` 注解自动装配 bean，要想把类标识成可用于 `@Autowired` 注解自动装配的 bean 的类,采用以下注解可实现：

- @**Component** ：通用的注解，可标注任意类为 Spring 组件。如果一个Bean不知道属于哪个层，可以使用@Component 注解标注。
- @**Repository** : 对应持久层即 Dao 层，主要用于数据库相关操作。
- @**Service** : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao层。
- @**Controller** : 对应 Spring MVC 控制层，主要用户接受用户请求并调用 Service 层返回数据给前端页面。





#### 参考资料

- https://blog.csdn.net/fuzhongmin05/article/details/73389779
- https://yemengying.com/2016/07/14/spring-bean-life-cycle/