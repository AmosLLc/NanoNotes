[TOC]

### JDBC 

#### JDBC流程

##### 1. 加载JDBC驱动程序

在连接数据库之前，首先要加载想要连接的数据库的驱动到 JVM（Java虚拟机），
这通过 java.lang.Class 类的静态方法 **forName**(String className) 实现。
例如：

```java
try{
    // 加载MySql的驱动类
    Class.forName("com.mysql.jdbc.Driver") ;
} catch(ClassNotFoundException e) {
    System.out.println("找不到驱动程序类 ，加载驱动失败！");
    e.printStackTrace() ;
}
```

成功加载后，会将 Driver 类的实例注册到 **DriverManager** 类中。

##### 2. 提供JDBC连接的URL

连接 URL 定义了连接数据库时的协议、子协议、数据源标识。

```
书写形式：协议：子协议：数据源标识
```

- 协议：在 JDBC 中总是以 jdbc 开始 

- 子协议：是桥连接的驱动程序或是数据库管理系统名称。

- 数据源标识：标记找到数据库来源的地址与连接端口。

例如：

```properties
jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=gbk;useUnicode=true
```

表示使用 Unicode 字符集。如果 characterEncoding 设置为 gb2312或GBK，本参数必须设置为 true 。characterEncoding=gbk：字符编码方式。

##### 3. 创建数据库的连接

要连接数据库，需要向 java.sql.**DriverManager **请求并获得 **Connection 对象**， 该对象就代表**一个数据库的连接**。

使用 DriverManager 的 **getConnectin**(String url , String username , String password ) 方法传入指定的欲连接的数据库的路径、数据库的用户名和 密码来获得。

例如：

```java
String url = "jdbc:mysql://localhost:3306/test" ;
String username = "root" ;
String password = "root" ;
try{
    Connection con = DriverManager.getConnection(url , username , password ) ;
}catch(SQLException se){
    System.out.println("数据库连接失败！");
    se.printStackTrace() ;
}
```

##### 4. 创建一个Statement

要执行 SQL 语句，必须获得 java.sql.**Statement** 实例，Statement 实例分为以下 3 种类型：

1、执行**静态 SQL** 语句。通常通过 **Statement** 实例实现。
2、执行**动态 SQL**语句。通常通过 **PreparedStatement** 实例实现。
3、执行数据库**存储过程**。通常通过 **CallableStatement** 实例实现。

具体的实现方式：

```java
Statement stmt = con.createStatement(); 
PreparedStatement pstmt = con.prepareStatement(sql); 
CallableStatement cstmt = con.prepareCall("XXXXXXXX");
```

##### 5. 执行SQL语句

**Statement** 接口提供了三种执行 SQL 语句的方法：**executeQuery 、executeUpdate 和 execute**

1、**ResultSet executeQuery(String sqlString)**：执行查询数据库的 SQL 语句 ，返回一个结果集（ResultSet）对象。
2、**int executeUpdate(String sqlString)**：用于执行 INSERT、UPDATE 或 DELETE 语句以及 SQL DDL 语句，如：CREATE TABLE 和 DROP TABLE 等。
3、**execute(sqlString)**：用于执行返回**多个结果集**、多个更新计数或二者组合的 语句。 

具体实现的代码：

```java
ResultSet rs = stmt.executeQuery(“SELECT * FROM …”); 
int rows = stmt.executeUpdate(“INSERT INTO …”); 
boolean flag = stmt.execute(String sql);
```

##### 6. 处理结果

两种情况：

1、执行**更新**返回的是本次操作**影响到的记录数**。
2、执行**查询**返回的结果是一个 **ResultSet 对象**。

**ResultSet** 包含符合 SQL 语句中条件的**所有行**，并且它通过一套 **get 方法**提供了对这些行中数据的访问。
使用结果集（ResultSet）对象的访问方法获取数据：（列是从左到右编号的，并且从列 1 开始）

```java
while(rs.next()){
    String name = rs.getString(“name”) ;
    String pass = rs.getString(1) ; // 此方法比较高效
}
```

##### 7. 关闭JDBC对象

操作完成以后要把所有使用的 JDBC 对象全都关闭，以释放 JDBC 资源，关闭顺序和声明顺序相反：
1、关闭记录集
2、关闭声明
3、关闭连接对象

```java
if(rs != null){ // 关闭记录集
    try{
        rs.close() ;
    }catch(SQLException e){
        e.printStackTrace() ;
    }
}
if(stmt != null){ // 关闭声明
    try{
        stmt.close() ;
    }catch(SQLException e){
        e.printStackTrace() ;
    }
}
if(conn != null){ // 关闭连接对象
    try{
        conn.close() ;
    }catch(SQLException e){
        e.printStackTrace() ;
    }
}
```

JDBC 用起来是比较繁杂的，很多步骤都是重复的，而且存在一些问题，所以才有了 MyBatis 这样的框架。