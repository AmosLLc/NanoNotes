[TOC]

### MySQL函数与存储过程

#### 函数

##### 1. 创建函数

学过的函数：LENGTH、SUBSTR、CONCAT 等
语法：

```mysql
CREATE FUNCTION 函数名(参数名 参数类型,...) RETURNS 返回类型
BEGIN
	函数体
END
```

##### 2. 调用函数

```mysql
SELECT 函数名（实参列表）
```

##### 3. MySQL函数内置

各个 DBMS 的函数都是不相同的，因此**不可移植**，以下主要是 MySQL 的函数。

|  函 数  |      说 明       |
| :-----: | :--------------: |
|  AVG()  | 返回某列的平均值 |
| COUNT() |  返回某列的行数  |
|  MAX()  | 返回某列的最大值 |
|  MIN()  | 返回某列的最小值 |
|  SUM()  |  返回某列值之和  |

**AVG**() 会忽略 **NULL** 行。

使用 DISTINCT 可以让汇总函数值汇总不同的值。

```sql
SELECT AVG(DISTINCT col1) AS avg_col
FROM mytable;
```

###### ① 文本处理

|   函数   |      说明      |   函数    |      说明      |
| :------: | :------------: | :-------: | :------------: |
|  LEFT()  |   左边的字符   |  RIGHT()  |   右边的字符   |
| LOWER()  | 转换为小写字符 |  UPPER()  | 转换为大写字符 |
| LTRIM()  | 去除左边的空格 |  RTRIM()  | 去除右边的空格 |
| LENGTH() |      长度      | SOUNDEX() |  转换为语音值  |

其中， **SOUNDEX()**  可以将一个字符串转换为描述其语音表示的字母数字模式。

```sql
SELECT *
FROM mytable
WHERE SOUNDEX(col1) = SOUNDEX('apple')
```

-----

###### ② 日期和时间处理


- 日期格式：YYYY-MM-DD
- 时间格式：HH:MM:SS

|     函 数     |             说 明              |
| :-----------: | :----------------------------: |
|   AddDate()   |    增加一个日期（天、周等）    |
|   AddTime()   |    增加一个时间（时、分等）    |
|   CurDate()   |          返回当前日期          |
|   CurTime()   |          返回当前时间          |
|    Date()     |     返回日期时间的日期部分     |
|  DateDiff()   |        计算两个日期之差        |
|  Date_Add()   |     高度灵活的日期运算函数     |
| Date_Format() |  返回一个格式化的日期或时间串  |
|     Day()     |     返回一个日期的天数部分     |
|  DayOfWeek()  | 对于一个日期，返回对应的星期几 |
|    Hour()     |     返回一个时间的小时部分     |
|   Minute()    |     返回一个时间的分钟部分     |
|    Month()    |     返回一个日期的月份部分     |
|     Now()     |       返回当前日期和时间       |
|   Second()    |      返回一个时间的秒部分      |
|    Time()     |   返回一个日期时间的时间部分   |
|    Year()     |     返回一个日期的年份部分     |

```sql
mysql> SELECT NOW();
```

```
2018-4-14 20:25:11
```

--------

###### ③ 数值处理

|  函数  |  说明  | 函数  |  说明  |
| :----: | :----: | :---: | :----: |
| SIN()  |  正弦  | COS() |  余弦  |
| TAN()  |  正切  | ABS() | 绝对值 |
| SQRT() | 平方根 | MOD() |  余数  |
| EXP()  |  指数  | PI()  | 圆周率 |
| RAND() | 随机数 |       |        |

---

#### 存储过程

##### 1. 概述

存储过程可以看成是对一系列 SQL 操作的**批处理**。

使用存储过程的好处：

- 代码封装，保证了一定的安全性；
- 代码复用；
- 由于是预先编译，因此具有很高的性能。

但是阿里巴巴的开发规范规定**别用**存储过程。

命令行中创建存储过程**==需要自定义分隔符==**，因为命令行是以 **; 为结束符**，而存储过程中也包含了**分号**，因此会错误把这部分分号当成是结束符，造成语法错误。

包含 **in、out 和 inout** 三种参数。

给变量赋值都需要用 select into 语句。

每次只能给**一个变量**赋值，不支持集合的操作。

存储过程含义：一组经过**预先编译**的SQL语句的集合
存储过程好处：

```mysql
1、提高了SQL语句的重用性，减少了开发程序员的压力
2、提高了效率
3、减少了传输次数
```

存储过程分类：

```mysql
1、无返回无参
2、仅仅带IN类型，无返回有参
3、仅仅带OUT类型，有返回无参
4、既带IN又带OUT，有返回有参
5、带INOUT，有返回有参
注意：IN、OUT、INOUT都可以在一个存储过程中带多个
```

##### 2. 创建存储过程

语法：

```mysql
CREATE PROCEDURE 存储过程名(IN|OUT|INOUT 参数名  参数类型,...)
BEGIN
	存储过程体
END
```

类似于方法：

```mysql
修饰符 返回类型 方法名(参数类型 参数名,...){
	方法体;
}
```

注意

```mysql
1、需要设置新的结束标记
DELIMITER 新的结束标记
示例：
DELIMITER $

CREATE PROCEDURE 存储过程名(IN|OUT|INOUT 参数名  参数类型,...)
BEGIN
	sql语句1;
	sql语句2;
END $

2、存储过程体中可以有多条SQL语句，如果仅仅一条SQL语句，则可以省略BEGIN END

3、参数前面的符号的意思
IN:该参数只能作为输入 （该参数不能做返回值）
OUT：该参数只能作为输出（该参数只能做返回值）
INOUT：既能做输入又能做输出
```

```mysql
delimiter //		# 自定义分隔符

create procedure myprocedure(out ret int)
    begin
        declare y int;
        select sum(col1)
        from mytable
        into y;
        select y*y into ret;
    end //

delimiter ;			# 换回分隔符
```

##### 3. 调用存储过程

```mysql
CALL 存储过程名(实参列表)
```

```mysql
call myprocedure(@ret);
select @ret;
```

##### 4. 游标

在存储过程中使用**游标**可以对**一个结果集**进行**移动遍历**。

游标主要用于**交互式应用**，其中用户需要对数据集中的任意行进行浏览和修改。

使用游标的四个步骤：

1. **声明游标**，这个过程**没有**实际检索出数据；
2. **打开游标**；
3. **取出数据**；
4. **关闭游标**；

```sql
delimiter //
create procedure myprocedure(out ret int)
    begin
        declare done boolean default 0;
        declare mycursor cursor for
        select col1 from mytable;
        # 定义了一个 continue handler，当 sqlstate '02000' 这个条件出现时，会执行 set done = 1
        declare continue handler for sqlstate '02000' set done = 1;

        open mycursor;

        repeat
            fetch mycursor into ret;
            select ret;
        until done end repeat;

        close mycursor;
    end //
 delimiter ;
```

---

#### 流程控制结构

##### 1. 系统变量

###### ① **全局变量**

作用域：针对于所有会话（连接）有效，但不能跨重启

```mysql
# 查看所有全局变量
SHOW GLOBAL VARIABLES;
# 查看满足条件的部分系统变量
SHOW GLOBAL VARIABLES LIKE '%char%';
# 查看指定的系统变量的值
SELECT @@global.autocommit;
# 为某个系统变量赋值
SET @@global.autocommit=0;
SET GLOBAL autocommit=0;
```

###### ② **会话变量**

作用域：针对于当前会话（连接）有效

```mysql
# 查看所有会话变量
SHOW SESSION VARIABLES;
# 查看满足条件的部分会话变量
SHOW SESSION VARIABLES LIKE '%char%';
# 查看指定的会话变量的值
SELECT @@autocommit;
SELECT @@session.tx_isolation;
# 为某个会话变量赋值
SET @@session.tx_isolation='read-uncommitted';
SET SESSION tx_isolation='read-committed';
```

##### 2. 自定义变量

###### ① **用户变量**

声明并初始化：

```mysql
SET @变量名=值;
SET @变量名:=值;
SELECT @变量名:=值;
```

赋值：

```mysql
方式一：一般用于赋简单的值
SET 变量名=值;
SET 变量名:=值;
SELECT 变量名:=值;
```


```mysql
方式二：一般用于赋表 中的字段值
SELECT 字段名或表达式 INTO 变量
FROM 表;
```

使用：

```mysql
SELECT @变量名;
```

###### ② **局部变量**

声明：

```mysql
DECLARE 变量名 类型 【DEFAULT 值】;
```

赋值：

```mysql
方式一：一般用于赋简单的值
SET 变量名 = 值;
SET 变量名:=值;
SELECT 变量名:=值;
```


```mysql
方式二：一般用于赋表 中的字段值
SELECT 字段名或表达式 INTO 变量
FROM 表;
```

使用：

```mysql
SELECT 变量名
```

##### 3. 分支

###### ① IF 函数

​	语法：IF (条件，值1，值2)
​	特点：可以用在任何位置

###### ② CASE 语句

语法：

```mysql
情况一：类似于SWITCH
CASE 表达式
WHEN 值1 THEN 结果1或语句1(如果是语句，需要加分号) 
WHEN 值2 THEN 结果2或语句2(如果是语句，需要加分号)
...
ELSE 结果n或语句n(如果是语句，需要加分号)
END 【CASE】（如果是放在BEGIN END中需要加上CASE，如果放在SELECT后面不需要）

情况二：类似于多重 IF
CASE 
WHEN 条件1 THEN 结果1或语句1(如果是语句，需要加分号) 
WHEN 条件2 THEN 结果2或语句2(如果是语句，需要加分号)
...
ELSE 结果n或语句n(如果是语句，需要加分号)
END 【CASE】（如果是放在BEGIN END中需要加上CASE，如果放在SELECT后面不需要）
```


特点：
	可以用在任何位置

###### ③ IF ELSEIF 语句

语法：

```mysql
IF 情况1 THEN 语句1;
ELSEIF 情况2 THEN 语句2;
...
ELSE 语句n;
END IF;
```

特点：
	**只能**用在 BEGIN END 中！

三者比较：应用场合

- IF 函数		简单双分支
- CASE  结构	等值判断 的多分支
- IF 结构		区间判断 的多分支

##### 4. 循环

语法：


```mysql
【标签：】WHILE 循环条件  DO
	循环体
END WHILE 【标签】;
```

特点：

- 只能放在 BEGIN END 里面
- 如果要搭配 LEAVE 跳转语句，需要使用标签，否则可以不用标签
- LEAVE 类似于 Java 中的 break 语句，跳出所在循环！！！











