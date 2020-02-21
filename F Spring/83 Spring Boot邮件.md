[TOC]

### Spring Boot邮件

#### 邮件任务

利用 Spring Boot 发送邮件，先发送到 QQ 服务器再转发给其他的。

![1563448228486](assets/1563448228486.png)

依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

配置信息

```properties
spring.mail.host=smtp.servie.com
spring.mail.username=用户名  // 发送方的邮箱
spring.mail.password=密码    // 对于qq邮箱而言 密码指的就是发送方的授权码 需要去邮箱申请
spring.mail.port=465
spring.mail.protocol=smtp
spring.mail.properties.mail.smtp.auth=true
# 是否用启用加密传送的协议验证项
spring.mail.properties.mail.smtp.ssl.enable=true
# 注意：在spring.mail.password处的值是需要在邮箱设置里面生成的授权码，这个不是真实的密码。
```

实现

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.FileSystemResource;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.stereotype.Component;
import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;
import java.io.File;

@Component("emailtool")
public class EmailTool {
    @Autowired
    private JavaMailSender javaMailSender;
    
    @Autowired
    private JavaMailSender mailSender;

    /**
    * 发送简单信息
    */
    @Override
    public void sendSimpleMail(String to, String subject, String content) throws MailException {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(from); // 邮件发送者
        message.setTo(to); // 邮件接受者
        message.setSubject(subject); // 主题
        message.setText(content); // 内容

        mailSender.send(message);
    }
    
    /**
    * 发送复杂邮件信息 可带附件啥的
    */
    public void sendSimpleMail(){
        MimeMessage message = null;
        try {
            message = javaMailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom("jiajinhao@dbgo.cn");
            helper.setTo("653484166@qq.com");
            helper.setSubject("标题：发送Html内容");

            StringBuffer sb = new StringBuffer();
            sb.append("<h1>大标题-h1</h1>")
                    .append("<p style='color:#F00'>红色字</p>")
                    .append("<p style='text-align:right'>右对齐</p>");
            helper.setText(sb.toString(), true);
            FileSystemResource fileSystemResource=new FileSystemResource(new File("D:\76678.pdf"))
            helper.addAttachment("电子发票"，fileSystemResource);
            javaMailSender.send(message);
        } catch (MessagingException e) {
            e.printStackTrace();
        }
    }

}
```







