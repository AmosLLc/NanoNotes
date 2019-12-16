[TOC]

### MySQL触发器

#### 触发器

触发器是与表有关的数据库对象，在**满足定义条件时触发**，并执行触发器中定义的语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性。

举个例子，比如你现在有两个表【用户表】和【日志表】，当一个用户被创建的时候，就需要在日志表中插入创建的 log 日志，如果在不使用触发器的情况下，你需要编写程序语言逻辑才能实现，但是如果你定义了一个触发器，触发器的作用就是当你在用户表中插入一条数据的之后帮你在日志表中插入一条日志信息。当然触发器并不是只能进行插入操作，还能执行修改，删除。

##### 创建触发器

创建触发器的语法如下：

```mysql
CREATE TRIGGER trigger_name trigger_time trigger_event ON tb_name FOR EACH ROW 
# trigger_stmt
# trigger_name：触发器的名称
# tirgger_time：触发时机，为BEFORE或者AFTER
# trigger_event：触发事件，为INSERT、DELETE或者UPDATE
# tb_name：表示建立触发器的表明，就是在哪张表上建立触发器
# trigger_stmt：触发器的程序体，可以是一条SQL语句或者是用BEGIN和END包含的多条语句
# 所以可以说MySQL创建以下六种触发器：
BEFORE INSERT, BEFORE DELETE, BEFORE UPDATE
AFTER INSERT, AFTER DELETE, AFTER UPDATE
```

其中，触发器名参数指要创建的触发器的名字.

BEFORE和AFTER参数指定了触发执行的时间，在事件之前或是之后。

FOR EACH ROW表示任何一条记录上的操作满足触发事件都会触发该触发器。

**创建有多个执行语句的触发器**

```mysql
CREATE TRIGGER 触发器名 BEFORE|AFTER 触发事件
ON 表名 FOR EACH ROW
BEGIN
    执行语句列表
END
```

其中，BEGIN 与 END 之间的执行语句列表参数表示需要执行的**多个**语句，不同语句用分号隔开。

**tips：**一般情况下，mysql默认是以 **;** 作为结束执行语句，与触发器中需要的分行起冲突，为解决此问题可用 **DELIMITER**，如：DELIMITER ||，可以将结束符号变成**||**， 当触发器创建完成后，可以用 **DELIMITER ;** 来将结束符号变成 **;** 。

```mysql
mysql> DELIMITER ||
mysql> CREATE TRIGGER demo BEFORE DELETE
    -> ON users FOR EACH ROW
    -> BEGIN
    -> INSERT INTO logs VALUES(NOW());
    -> INSERT INTO logs VALUES(NOW());
    -> END
    -> ||
Query OK, 0 rows affected (0.06 sec)

mysql> DELIMITER ;
```

上面的语句中，开头将结束符号定义为||，中间定义一个触发器，一旦有满足条件的删除操作。就会执行 BEGIN 和 END 中的语句，接着使用 || 结束。最后使用DELIMITER ; 将结束符号**还原**。



#### 其他

触发器会在**某个表**执行以下语句时而**自动执行**：==DELETE、INSERT、UPDATE==。

触发器必须指定在语句执行**之前**还是**之后**自动执行，之前执行使用 **BEFORE** 关键字，之后执行使用 **AFTER** 关键字。**BEFORE 用于数据验证和净化，AFTER 用于审计跟踪**，将修改记录到另外一张表中。

INSERT 触发器包含一个名为 NEW 的虚拟表。

```sql
CREATE TRIGGER mytrigger AFTER INSERT ON mytable
FOR EACH ROW SELECT NEW.col into @result;

SELECT @result; -- 获取结果
```

DELETE 触发器包含一个名为 OLD 的虚拟表，并且是只读的。

UPDATE 触发器包含一个名为 NEW 和一个名为 OLD 的虚拟表，其中 NEW 是可以被修改的，而 OLD 是只读的。

MySQL 不允许在触发器中使用 CALL 语句，也就是不能调用存储过程。

