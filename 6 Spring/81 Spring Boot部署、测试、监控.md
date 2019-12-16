[TOC]

## Spring Boot部署、测试、监控



### 1 Spring Boot部署

#### Spring Boot热部署

依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

修改后 使用Ctrl + F9 进行快捷编译即可。





### 2 Spring Boot测试







### 3 Spring Boot监控

通过引入spring-boot-starter-actuator，可以使用Spring Boot为我们提供的准生产环境下的应用监控和管理功能。我们可以通过HTTP，JMX，SSH协议来进行操作，自动得到审计、健康及指标信息等

步骤：

- 引入spring-boot-starter-actuator

- 通过http方式访问监控端点

- 可进行shutdown（POST 提交，此端点默认关闭）

#### 监控和管理端点

| **端点名**   | **描述**                    |
| ------------ | --------------------------- |
| *autoconfig* | 所有自动配置信息            |
| auditevents  | 审计事件                    |
| beans        | 所有Bean的信息              |
| configprops  | 所有配置属性                |
| dump         | 线程状态信息                |
| env          | 当前环境信息                |
| health       | 应用健康状况                |
| info         | 当前应用信息                |
| metrics      | 应用的各项指标              |
| mappings     | 应用@RequestMapping映射路径 |
| shutdown     | 关闭当前应用（默认关闭）    |
| trace        | 追踪信息（最新的http请求）  |





