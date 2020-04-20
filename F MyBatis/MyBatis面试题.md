[TOC]

### MyBatis基础

#### MyBatis 编程步骤

1. 创建 SqlSessionFactory 对象。
2. 通过 SqlSessionFactory 获取 SqlSession 对象。
3. 通过 SqlSession 获得 Mapper 代理对象。
4. 通过 Mapper 代理对象，执行数据库操作。
5. 执行成功，则使用 SqlSession 提交事务。
6. 执行失败，则使用 SqlSession 回滚事务。
7. 最终，关闭会话。

#### `#{}` 和 `${}` 的区别是什么？

`${}` 是 Properties 文件中的**变量占位符**，它可以用于 XML 标签属性值和 SQL 内部，属于**字符串替换**。例如将 `${driver}` 会被静态替换为 `com.mysql.jdbc.Driver` ：

```XML
<dataSource type="UNPOOLED">
    <property name="driver" value="${driver}"/>
    <property name="url" value="${url}"/>
    <property name="username" value="${username}"/>
</dataSource>
```

`${}` 也可以对传递进来的参数**原样拼接**在 SQL 中。代码如下：

```XML
<select id="getSubject3" parameterType="Integer" resultType="Subject">
    SELECT * FROM subject
    WHERE id = ${id}
</select>
```

生产环境下，**不推荐**这么做。因为，可能有 **SQL 注入**的风险。

------

`#{}` 是 SQL 的**参数占位符**，Mybatis 会将 SQL 中的 `#{}` 替换为 `?` 号，在 SQL 执行前会使用 **PreparedStatement** 的参数设置方法，按序给 SQL 的 `?` 号占位符设置参数值，比如 `ps.setInt(0, parameterValue)` 。 所以，`#{}` 是**预编译处理**，可以**有效防止 SQL 注入**，提高系统安全性。

------

另外，`#{}` 和 `${}` 的取值方式非常方便。例如：`#{item.name}` 的取值方式，为使用反射从参数对象中，获取 `item` 对象的 `name` 属性值，相当于 `param.getItem().getName()` 。

#### 当实体类中的属性名和表中的字段名不一样 ，怎么办？

第一种， 通过在查询的 SQL 语句中定义字段名的别名，让字段名的别名和实体类的属性名一致。代码如下：

```xml
<select id="selectOrder" parameterType="Integer" resultType="Order"> 
    SELECT order_id AS id, order_no AS orderno, order_price AS price 
    FROM orders 
    WHERE order_id = #{id}
</select>
```

- 这里，艿艿还有几点建议：
  - 1、数据库的关键字，统一使用大写，例如：`SELECT`、`AS`、`FROM`、`WHERE` 。
  - 2、每 5 个查询字段换一行，保持整齐。
  - 3、`,` 的后面，和 `=` 的前后，需要有空格，更加清晰。
  - 4、`SELECT`、`FROM`、`WHERE` 等，单独一行，高端大气。

------

第二种，是第一种的特殊情况。大多数场景下，数据库字段名和实体类中的属性名差，主要是前者为**下划线风格**，后者为**驼峰风格**。在这种情况下，可以直接配置如下，实现自动的下划线转驼峰的功能。

```
<setting name="logImpl" value="LOG4J"/>
    <setting name="mapUnderscoreToCamelCase" value="true" />
</settings>
```

😈 也就说，约定大于配置。非常推荐！

------

第三种，通过 `` 来映射字段名和实体类属性名的一一对应的关系。代码如下：

```
<resultMap type="me.gacl.domain.Order" id=”OrderResultMap”> 
    <!–- 用 id 属性来映射主键字段 -–> 
    <id property="id" column="order_id"> 
    <!–- 用 result 属性来映射非主键字段，property 为实体类属性名，column 为数据表中的属性 -–> 
    <result property="orderNo" column ="order_no" /> 
    <result property="price" column="order_price" /> 
</resultMap>

<select id="getOrder" parameterType="Integer" resultMap="OrderResultMap">
    SELECT * 
    FROM orders 
    WHERE order_id = #{id}
</select>
```

- 此处 `SELECT *` 仅仅作为示例只用，实际场景下，千万千万千万不要这么干。用多少字段，查询多少字段。
- 相比第一种，第三种的**重用性**会一些。

#### XML 映射文件中，除了常见的 select | insert | update | delete标 签之外，还有哪些标签？

如下部分，可见 [《MyBatis 文档 —— Mapper XML 文件》](http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html) ：

- ```
  <cache />
  ```

   

  标签，给定命名空间的缓存配置。

  - `` 标签，其他命名空间缓存配置的引用。

- `` 标签，是最复杂也是最强大的元素，用来描述如何从数据库结果集中来加载对象。

- ~~`` 标签，已废弃！老式风格的参数映射。内联参数是首选,这个元素可能在将来被移除，这里不会记录。~~

- ```
  <sql />
  ```

   

  标签，可被其他语句引用的可重用语句块。

  - `` 标签，引用 `` 标签的语句。

- `` 标签，不支持自增的主键生成策略标签。

如下部分，可见 [《MyBatis 文档 —— 动态 SQL》](http://www.mybatis.org/mybatis-3/zh/dynamic-sql.html) ：

- ``
- ``、``、``
- ``、``、``
- ``
- ``

#### Mybatis 动态 SQL 是做什么的？都有哪些动态 SQL ？能简述一下动态 SQL 的执行原理吗？

- Mybatis 动态 SQL ，可以让我们在 XML 映射文件内，以 XML 标签的形式编写动态 SQL ，完成逻辑判断和动态拼接 SQL 的功能。
- Mybatis 提供了 9 种动态 SQL 标签：``、``、``、``、``、``、``、``、`` 。
- 其执行原理为，使用 **OGNL** 的表达式，从 SQL 参数对象中计算表达式的值，根据表达式的值动态拼接 SQL ，以此来完成动态 SQL 的功能。

如上的内容，更加详细的话，请看 [《MyBatis 文档 —— 动态 SQL》](http://www.mybatis.org/mybatis-3/zh/dynamic-sql.html) 文档。

#### 最佳实践中，通常一个 XML 映射文件，都会写一个 Mapper 接口与之对应。请问，这个 Mapper 接口的工作原理是什么？Mapper 接口里的方法，参数不同时，方法能重载吗？

Mapper 接口，对应的关系如下：

- 接口的全限名，就是映射文件中的 `"namespace"` 的值。
- 接口的方法名，就是映射文件中 MappedStatement 的 `"id"` 值。
- 接口方法内的参数，就是传递给 SQL 的参数。

Mapper 接口是没有实现类的，当调用接口方法时，接口全限名 + 方法名拼接字符串作为 key 值，可唯一定位一个对应的 MappedStatement 。举例：`com.mybatis3.mappers.StudentDao.findStudentById` ，可以唯一找到 `"namespace"` 为 `com.mybatis3.mappers.StudentDao` 下面 `"id"` 为 `findStudentById` 的 MappedStatement 。

总结来说，在 Mybatis 中，每一个 ``、``、``、`` 标签，都会被解析为一个 MappedStatement 对象。

另外，Mapper 接口的实现类，通过 MyBatis 使用 **JDK Proxy** 自动生成其代理对象 Proxy ，而代理对象 Proxy 会拦截接口方法，从而“调用”对应的 MappedStatement 方法，最终执行 SQL ，返回执行结果。整体流程如下图：

