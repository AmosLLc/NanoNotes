[TOC]

### Mybatis高级

#### 缓存

MyBatis 提供**查询缓存**，用于减轻数据压力，提高数据库性能。

MyBatis 提供**一级缓存，和二级缓存**。

**一级缓存是 SqlSession 级别的缓存**。在操作数据库时需要构造 sqlSession 对象，在对象中有一个**数据结构（HashMap）**用于**存储缓存数据**。不同的 sqlSession 之间的缓存数据区域（HashMap）是**互相不影响**的。

**二级缓存是 mapper 级别的缓存**，多个 SqlSession 去操作同一个 Mapper 的 sql 语句，多个 SqlSession 可以**共用二级缓存**，二级缓存是**跨 SqlSession** 的。

为什么要用缓存？

如果缓存中有数据就不用从数据库中获取，大大提高系统性能。

##### 1. 一级缓存

但是在默认的情况下， 只开启一级缓存（一级缓存是对同一个 SqlSession 而言的）。

同一个 `SqlSession` 对象， 在参数和 SQL 完全一样的情况下， 只执行一次 SQL 语句（如果缓存没有过期）。也就是只有在**参数和 SQL 完全一样**的情况下， 才会有这种情况。

```java
@Test
public void oneSqlSession() {
    SqlSession sqlSession = null;
    try {
        sqlSession = sqlSessionFactory.openSession();
        StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
        // 执行第一次查询
        List<Student> students = studentMapper.selectAll();
        for (int i = 0; i < students.size(); i++) {
            System.out.println(students.get(i));
        }
        System.out.println("=============开始同一个 Sqlsession 的第二次查询============");
        // 同一个 sqlSession 进行第二次查询
        List<Student> stus = studentMapper.selectAll();
        Assert.assertEquals(students, stus);
        for (int i = 0; i < stus.size(); i++) {
            System.out.println("stus:" + stus.get(i));
        }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        if (sqlSession != null) {
            sqlSession.close();
        }
    }
}
```

在以上的代码中， 进行了两次查询， 使用相同的 `SqlSession`, **第一次**查询发送了 SQL 语句， 后返回了结果；

**第二次**查询没有发送 SQL 语句， 直接从内存中获取了结果。而且两次结果输入一致， 同时断言两个对象相同也通过。

```java
@Test
public void differSqlSession() {
    SqlSession sqlSession = null;
    SqlSession sqlSession2 = null;
    try {
        sqlSession = sqlSessionFactory.openSession();

        StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
        // 执行第一次查询
        List<Student> students = studentMapper.selectAll();
        for (int i = 0; i < students.size(); i++) {
            System.out.println(students.get(i));
        }
        System.out.println("=============开始不同 Sqlsession 的第二次查询============");
        // 从新创建一个 sqlSession2 进行第二次查询
        sqlSession2 = sqlSessionFactory.openSession();
        StudentMapper studentMapper2 = sqlSession2.getMapper(StudentMapper.class);
        List<Student> stus = studentMapper2.selectAll();
        // 不相等
        Assert.assertNotEquals(students, stus);
        for (int i = 0; i < stus.size(); i++) {
            System.out.println("stus:" + stus.get(i));
        }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        if (sqlSession != null) {
            sqlSession.close();
        }
        if (sqlSession2 != null) {
            sqlSession2.close();
        }
    }
}
```

在代码中， 分别使用 **sqlSession** 和 **sqlSession2** 进行了相同的查询。

从日志中可以看到两次查询都分别从数据库中取出了数据。 虽然结果相同， 但两个是**不同的对象**。

**刷新缓存**

**刷新缓存是清空这个 SqlSession 的所有缓存， 不单单是某个键。**

```java
@Test
public void sameSqlSessionNoCache() {
    SqlSession sqlSession = null;
    try {
        sqlSession = sqlSessionFactory.openSession();
        StudentMapper studentMapper = sqlSession.getMapper(StudentMapper.class);
        // 执行第一次查询
        Student student = studentMapper.selectByPrimaryKey(1);
        System.out.println("=============开始同一个 Sqlsession 的第二次查询============");
        // 同一个 sqlSession 进行第二次查询
        Student stu = studentMapper.selectByPrimaryKey(1);
        Assert.assertEquals(student, stu);
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        if (sqlSession != null) {
            sqlSession.close();
        }
    }
}
```

如果是以上， 没什么不同， 结果还是第二个不发 SQL 语句。

在此， 做一些修改， 在 **StudentMapper.xml** 中， 添加

> **flushCache=“true”**

```xml
<select id="selectByPrimaryKey" flushCache="true" parameterType="java.lang.Integer" resultMap="BaseResultMap">
    select
    <include refid="Base_Column_List" />
    from student
    where student_id=#{id, jdbcType=INTEGER}
</select>
```

第一次， 第二次都发送了 SQL 语句， 同时， 断言两个对象相同出错。

**总结**

- 在同一个 SqlSession 中, Mybatis 会把**执行的方法和参数通过算法生成缓存的键值**， 将**键值和结果存放在一个 Map** 中， 如果后续的键值一样， 则直接从 Map 中获取数据；

- 不同的 SqlSession 之间的缓存是**相互隔离**的；

- 用一个 SqlSession， 可以**通过配置使得在查询前清空缓存**；

- 任何的 UPDATE, INSERT, DELETE 语句都会清空缓存。