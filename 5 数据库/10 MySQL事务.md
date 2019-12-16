[TOC]

### MySQL事务

#### 概述

- 含义通过一组逻辑操作单元（一组DML——sql 语句），将数据从一种状态切换到另外一种状态。

- 在 MySQL 中只有使用了 Innodb 数据库引擎的数据库或表才支持事务。
- 事务处理可以用来维护数据库的完整性，保证成批的 SQL 语句要么全部执行，要么全部不执行。
- 事务用来管理 insert, update, delete 语句。 

#### 事务四大特征

一般来说，事务是必须满足4个条件（ACID）：原子性（Atomicity，或称不可分割性）、一致性（Consistency）、隔离性（Isolation，又称独立性）、持久性（Durability）。

- 原子性：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
- 一致性：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。(比如：A 向 B 转账，不可能 A 扣了钱，B 却没有收到)。
- 隔离性：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。**事务隔离**分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。（比如：A 正在从一张银行卡里面取钱，在 A 取钱的过程中，B 不能向这张银行卡打钱）。
- 持久性：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。



#### 事务的分类：

隐式事务，没有明显的开启和结束事务的标志

```mysql
比如
INSERT、UPDATE、DELETE 语句本身就是一个事务
```


显式事务，具有明显的开启和结束事务的标志

```mysql
1、开启事务
	取消自动提交事务的功能
2、编写事务的一组逻辑操作单元（多条sql语句）
	INSERT
	UPDATE
	DELETE
3、提交事务或回滚事务
```

----

#### 使用到的关键字

```mysql
SET AUTOCOMMIT = 0;
START TRANSACTION;
COMMIT;
ROLLBACK;

SAVEPOINT  断点
COMMIT TO 断点
ROLLBACK TO 断点
```

----

#### 事务的隔离级别

事务并发问题如何发生？

	当多个事务同时操作同一个数据库的相同数据时

事务的并发问题有哪些？

	脏读：一个事务读取到了另外一个事务未提交的数据
	不可重复读：同一个事务中，多次读取到的数据不一致
	幻读：一个事务读取数据时，另外一个事务进行更新，导致第一个事务读取到了没有更新的数据

如何避免事务的并发问题？通过设置事务的隔离级别。

```mysql
1、READ UNCOMMITTED
2、READ COMMITTED 可以避免脏读
3、REPEATABLE READ 可以避免脏读、不可重复读和一部分幻读
4、SERIALIZABLE可以避免脏读、不可重复读和幻读
```

四个隔离级别及其解决的问题

|     隔离级别     | 脏读 | 不可重复读 | 幻读 |
| :--------------: | :--: | :--------: | :--: |
| Read uncommitted |  ×   |     ×      |  ×   |
|  Read committed  |  √   |     ×      |  ×   |
| Repeatable read  |  √   |     √      |  ×   |
|   Serializable   |  √   |     √      |  √   |

设置隔离级别：

```mysql
SET SESSION|GLOBAL TRANSACTION ISOLATION LEVEL 隔离级别名;
```

查看隔离级别：

```mysql
SELECT @@tx_isolation;
```

```mysql
SHOW VARIABLES LIKE 'autocommit';
SHOW ENGINES;

# 1.演示事务的使用步骤
# 开启事务
SET autocommit = 0;
START TRANSACTION;
# 编写一组事务的语句
UPDATE account SET balance = 1000 WHERE username='张无忌';
UPDATE account SET balance = 1000 WHERE username='赵敏';

# 结束事务
ROLLBACK;
# COMMIT;

SELECT * FROM account;

# 2.演示事务对于delete和truncate的处理的区别
SET autocommit = 0;
START TRANSACTION;

DELETE FROM account;
ROLLBACK;

# 3.演示SAVEPOINT的使用
SET autocommit = 0;
START TRANSACTION;
DELETE FROM account WHERE id = 25;
SAVEPOINT a;	# 设置保存点
DELETE FROM account WHERE id = 28;
ROLLBACK TO a;	# 回滚到保存点

SELECT * FROM account;
```



#### 事务提交、回滚

```mysql
mysql> start transaction;  # 手动开启事务
mysql> insert into t_user(name) values('pp');
mysql> commit;  # commit之后即可改变底层数据库数据
mysql> select * from t_user;
+----+------+
| id | name |
+----+------+
|  1 | jay  |
|  2 | man  |
|  3 | pp   |
+----+------+
3 rows in set (0.00 sec)

mysql> start transaction;
mysql> insert into t_user(name) values('yy');
mysql> rollback;
mysql> select * from t_user;
+----+------+
| id | name |
+----+------+
|  1 | jay  |
|  2 | man  |
|  3 | pp   |
+----+------+
3 rows in set (0.00 sec)
```









