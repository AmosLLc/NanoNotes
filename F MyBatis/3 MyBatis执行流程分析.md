[TOC]

#### MyBatis执行流程

##### 1. 配置文件解析Configuration

讲解解析流程之前先回顾一下 MyBatis 中配置文件的结构：

**mybatis-config.xml**

```xml
<configuration>
    <properties/>
    <settting/>
    <typeHandlers/>
    <..../>
    <mappers/>
</configuration>
```

\<mappers/> 对应 mapper 文件。

如：mybatis-mapper.xml。

```xml
<mapper > 
    <cache/>
    <resultMap/>
    <select/>  
    <update/> 
    <delete/> 
    <insert/> 
</mapper>
```

配置文件的解析流程即是将上述 XML 描述元素转换成对应的 **JAVA 对象**过程，其最终转换对像及其关系如下图：



**配置元素解析构建器**过程：

解析 XML：

```java
>org.apache.ibatis.builder.xml.XMLConfigBuilder
 >org.apache.ibatis.builder.xml.XMLMapperBuilder
  >org.apache.ibatis.builder.xml.XMLStatementBuilder
   >org.apache.ibatis.builder.SqlSourceBuilder
    >org.apache.ibatis.scripting.xmltags.XMLScriptBuilder
```

解析注解：

```java
>org.apache.ibatis.builder.annotation.MapperAnnotationBuilder
```

 







