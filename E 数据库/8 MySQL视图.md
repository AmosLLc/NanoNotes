[TOC]

### MySQL视图

含义：理解成一张**虚拟的表**。MySQL5.1版本出现的新特性，是通过表动态生成的数据

视图和表的区别：两者使用方式完全相同，不过表需要占用物理空间，而视图不占用物理空间，仅仅保存的是SQL逻辑。
视图的好处：

- 简化复杂的 SQL 操作，比如复杂的连接；
- 只使用实际表的一**部分数据**；
- 通过只给用户**访问视图的权限**，保证数据的安全性；
- 更改数据格式和表示。



#### 视图的作用

- 测试表: user 有id，name，age，sex 字段

- 测试表: goods 有id，name，price 字段

- 测试表: ug 有 id，userid，goodsid 字段

**作用一：** 提高了重用性，就像一个函数。如果要频繁获取 user 的 name 和 goods 的 name。就应该使用以下sql语言。示例：

 ```mysql
select a.name as username, b.name as goodsname from user as a, goods as b, ug as c where a.id = c.userid and c.goodsid = b.id;
 ```

但有了视图就不一样了，创建视图 **other**。示例

 ```mysql
create view other as select a.name as username, b.name as goodsname from user as a, goods as b, ug as c where a.id = c.userid and c.goodsid = b.id;
 ```

创建好视图后，就可以这样获取 user 的 name 和 goods 的 name。示例：

 ```mysql
select * from other;
 ```

**作用二：**对数据库重构，却不影响程序的运行。假如因为某种需求，需要将 user 拆成表 usera 和表 userb，该两张表的结构如下：

- 测试表: usera 有 id，name，age 字段
- 测试表: userb 有 id，name，sex 字段

这时如果使用 sql 语句：select * from user; 那就会提示该表**不存在**，这时该如何解决呢。解决方案：创建视图。以下 sql 语句创建视图：

```mysql
create view user as select a.name,a.age,b.sex from usera as a, userb as b where a.name = b.name;
```

以上假设 name 都是唯一的。此时使用 sql 语句：select * from user; 就不会报错。这就实现了更改数据库结构，不更改脚本程序的功能了。

**作用三：** 提高了安全性能。可以对**不同的用户，设定不同的视图**。例如：某用户只能获取 user 表的 name 和 age数据，不能获取 sex 数据。则可以这样创建视图。示例如下：

 ```mysql
create view other as select a.name, a.age from user as a;
 ```

​    这样的话，使用sql语句：select * from other; 最多就只能获取 name 和 age 的数据，其他的数据就获取不了了。

**作用四：** 让数据更加清晰。想要什么样的数据，就创建什么样的视图。

----

#### 视图的创建

```mysql
语法：
CREATE VIEW  视图名
AS
查询语句;
```

-----

#### 视图的增删改查

```mysql
1、查看视图的数据 ★
SELECT * FROM my_v4;
SELECT * FROM my_v1 WHERE last_name = 'Partners';

2、插入视图的数据
INSERT INTO my_v4(last_name,department_id) VALUES('虚竹',90);

3、修改视图的数据

UPDATE my_v4 SET last_name = '梦姑' WHERE last_ame = '虚竹';

4、删除视图的数据
DELETE FROM my_v4;
```

----

#### 某些视图不能更新

```mysql
包含以下关键字的SQL语句：分组函数、DISTINCT、GROUP BY、HAVING、UNION或者UNION ALL
常量视图
SELECT中包含子查询
JOIN
FROM一个不能更新的视图
WHERE子句的子查询引用了FROM子句中的表
```

----

#### 视图逻辑的更新

```mysql
# 方式一：
CREATE OR REPLACE VIEW test_v7
AS
SELECT last_name FROM employees
WHERE employee_id>100;

# 方式二:
ALTER VIEW test_v7
AS
SELECT employee_id FROM employees;

SELECT * FROM test_v7;
```

----

#### 视图的删除

```mysql
DROP VIEW test_v1,test_v2,test_v3;
```

#### 视图结构的查看

```mysql
DESC test_v7;
SHOW CREATE VIEW test_v7;
```







