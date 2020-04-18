[TOC]

### MySQL视图

#### 视图概述

含义：理解成一张**虚拟的表**。MySQL5.1版本出现的新特性，是通过表**动态生成**的数据。

视图和表的区别：两者使用方式**完全相同**，不过表需要占用物理空间，而视图不占用物理空间，仅仅保存的是SQL逻辑。

视图的好处：

- 简化复杂的 SQL 操作，比如复杂的连接；
- 只使用实际表的一**部分数据**；
- 通过只给用户**访问视图的权限**，保证数据的安全性；
- 更改**数据格式和表示**。

---

#### 视图的作用

- 测试表: user 有id，name，age，sex 字段

- 测试表: goods 有id，name，price 字段

- 测试表: ug 有 id，userid，goodsid 字段

**作用一：** 提高了**重用性**，就像一个函数。如果要**频繁获取** user 的 name 和 goods 的 name。就应该使用以下 sql 语言。示例：

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

**作用二：**对数据库**重构**，却不影响程序的运行。假如因为某种需求，需要将 user 拆成表 usera 和表 userb，该两张表的结构如下：

- 测试表: usera 有 id，name，age 字段
- 测试表: userb 有 id，name，sex 字段

这时如果使用 sql 语句：select * from user; 那就会提示该表**不存在**，这时该如何解决呢。解决方案：**创建视图**。以下 sql 语句创建视图：

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

#### 视图的基本操作

**查询语句**放在 AS 后面。

```mysql
# 创建视图
CREATE VIEW  视图名
AS
查询语句;
# 删除视图
DROP VIEW test1, test2;
# 查看视图结构
DESC test1;
SHOW CREATE VIEW test1;
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

如果视图中数据是来自于**一个表**时，修改视图中的数据，表数据会**更新**。而且修改表中数据时，对应视图也会**更新**。但是如果视图数据来源于**两个表**时，修改视图数据时会报错，**无法**修改。具体见下面。

```mysql
包含以下关键字的SQL语句：分组函数、DISTINCT、GROUP BY、HAVING、UNION或者UNION ALL
常量视图
SELECT中包含子查询
JOIN
FROM一个不能更新的视图
WHERE子句的子查询引用了FROM子句中的表
```



### MySQL触发器

#### 概述

触发器是与表有关的数据库对象，在**满足定义条件时触发**，并**执行触发器中定义的语句**集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性。

举个例子，比如你现在有两个表【用户表】和【日志表】，当一个用户被创建的时候，就需要在日志表中插入创建的 log 日志，如果在不使用触发器的情况下，你需要**编写程序语言**逻辑才能实现，但是如果你定义了一个触发器，触发器的作用就是当你在用户表中插入一条数据的之后帮你在日志表中插入一条日志信息。当然触发器并不是只能进行**插入**操作，还能执行**修改，删除。**

#### 创建触发器

创建触发器的语法如下：

```mysql
CREATE TRIGGER trigger_name trigger_time trigger_event ON tb_name FOR EACH ROW 
# trigger_stmt
# trigger_name：触发器的名称
# tirgger_time：触发时机，为BEFORE或者AFTER
# trigger_event：触发事件，为INSERT、DELETE或者UPDATE
# tb_name：表示建立触发器的表明，就是在哪张表上建立触发器
# trigger_stmt：触发器的程序体，可以是一条SQL语句或者是用BEGIN和END包含的多条语句
```

```mysql
# 所以可以说MySQL根据触发时机创建以下六种触发器：
BEFORE INSERT, BEFORE DELETE, BEFORE UPDATE
AFTER INSERT, AFTER DELETE, AFTER UPDATE
```

其中，触发器名参数指要创建的触发器的名字.

BEFORE 和 AFTER 参数指定了触发**执行的时间**，在事件**之前**或是**之后**。

FOR EACH ROW 表示**任何一条记录**上的操作满足触发事件都会触发该触发器。

**创建有多个==执行语句==的触发器**

```mysql
CREATE TRIGGER 触发器名 BEFORE|AFTER 触发事件
ON 表名 FOR EACH ROW
BEGIN
    执行语句列表
END
```

其中，**BEGIN 与 END** 之间的执行语句列表参数表示需要执行的**多个**语句，不同语句用**分号**隔开。

**tips：**一般情况下，MySQL 默认是以 **;** 作为结束执行语句，与触发器中需要的**分行**起冲突，为解决此问题可用 **DELIMITER**，如：DELIMITER ||，可以将结束符号变成 **||**， 当触发器创建完成后，可以用 **DELIMITER ;** 来将结束符号变成 **;** 。这与存储过程的使用类似。

```mysql
mysql> DELIMITER ||		# 修改默认的结束符号
mysql> CREATE TRIGGER demo BEFORE DELETE
    -> ON users FOR EACH ROW		# 每一行都观测
    -> BEGIN
    -> INSERT INTO logs VALUES(NOW());	# BEGIN 与 END 中间是触发时执行的逻辑
    -> INSERT INTO logs VALUES(NOW());
    -> END
    -> ||
Query OK, 0 rows affected (0.06 sec)

mysql> DELIMITER ;
```

上面的语句中，开头将结束符号定义为 ||，中间定义一个触发器，一旦有满足条件的**删除操作**。就会执行 BEGIN 和 END 中的语句，接着**使用 || 结束**。最后使用DELIMITER ; 将结束符号**还原**。

---

#### 其他

触发器会在**某个表**执行以下语句时而**自动执行**：**==DELETE、INSERT、UPDATE==**。

触发器必须指定在语句执行**之前**还是**之后**自动执行，之前执行使用 **BEFORE** 关键字，之后执行使用 **AFTER** 关键字。**BEFORE 用于数据验证和净化，AFTER 用于审计跟踪**，将修改记录到另外一张表中。

INSERT 触发器包含一个名为 **NEW 的虚拟表**。

```sql
CREATE TRIGGER mytrigger AFTER INSERT ON mytable
FOR EACH ROW SELECT NEW.col into @result;

SELECT @result; -- 获取结果
```

**DELETE** 触发器包含一个名为 OLD 的虚拟表，并且是**只读**的。

**UPDATE** 触发器包含一个名为 NEW 和一个名为 OLD 的虚拟表，其中 NEW 是可以被修改的，而 OLD 是只读的。

MySQL 不允许在触发器中使用 CALL 语句，也就是**不能调用存储过程**。





