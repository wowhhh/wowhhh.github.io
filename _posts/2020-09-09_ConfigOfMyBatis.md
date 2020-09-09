---
layout: post
title:  "那些学完不用就后悔的MyBatis配置"
tags:   MyBatis
date:   2020-09-09 21:00:00 +0800
categories: [MyBatis]

---



#### 有哪些配置？

- Properties
- settings
- typeAliases
- environments
- mappers

MyBatis的配置项顺序不能颠倒

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--核心配置文件-->
<configuration>
<!--    配置多个环境-->
    <environments default="development">
<!--        用于开发 id也可以设置test等-->
        <environment id="development">
            <!--数据库配置-->
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
<!--                驱动-->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
<!--                连接数据库的url-->
                <property name="url" value="jdbc:mysql://127.0.0.1:3306/mybatis?serverTimezone=UTC&amp;
useUnicode=true;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="neu406"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="UserMapper.xml"/>
    </mappers>
</configuration>
```

#### 环境配置（Environment）

##### 作用

配置数据库

##### 主要配置参数

- 默认使用的环境

  ```environments```中使用的```default="development"``` ，如果说environment中又ID为test，当default设置为development的时候，就默认使用了test配置的环境了。

- 每个environment元素定义的环境ID

  ```environments```中使用的```id="development"``` 

- 事务管理器的配置

  ```xml
  <transactionManager type="JDBC"/>
  ```

  上面配置的**JDBC**（默认）类型的事务管理器，还有一种是```MANAGED```

  指定为JDBC类型的时候会使用JDBC管理事务机制。

  指定为MANAGED类型的时候会使用web容器管理事务。MANAGED这个配置基本没做什么。

- 数据源的配置

  用来连接数据库，类似以前的dbcp、c3p0和之后会学的，druid等数据源。做的功能都是连接数据库。
  
  ```xml
  <dataSource type="POOLED">
  ...
</dataSource>
  ```

  用来配置数据源类型，有三种可选 **UNPOOLED 、 POOLED 、 JNDI**

  - UNPOOLED（无连接池）

    这个数据源的实现会每次请求时打开和关闭连接。

    对那些数据库连接可用性要求不高的简单应用程序来说，是一个很好的选择。

  - **POOLED**（默认）
  
    这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。 这种处理方式很流行，能使并发 Web 应用快速响应请求。
  
  - JNDI
  

综上，MyBatis的默认管理器是JDBC，默认连接池是POOLED。要学会配置多套运行环境，也就是配置多个ID。

#### Properties

##### 作用

可以给系统配置一些运行参数，这些参数配置在XML中或者是配置在properties中，而不是配置在Java代码中，方便了修改，不需要重新编译。

可以通过Properties属性用来实现引用配置文件。

这些属性可以在外部进行配置，并可以进行动态替换。你既可以在典型的 Java 属性文件中配置这些属性，也可以在 properties 元素的子元素中设置。

##### 如何使用

编写一个配置文件：

```properties
driver = com.mysql.jdbc.Driver
url = jdbc:mysql://127.0.0.1:3306/mybatis?serverTimezone=UTC&amp;useUnicode=true;characterEncoding=UTF-8
username = root
password = neu406
```

在核心配置文件中引入：

```xml
<!--    引入配置文件-->
    <properties resource="db.properties">
<!--        暂时不写属性，properties里面又-->
    </properties>
```

如何取用

```xml
                <property name="driver" value="${driver}"/>
<!--                连接数据库的url-->
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
```

最后的运行结果和以前是相同的。

除了上面的方式外，还可以在properties元素的子元素里面设置。

```xml
<properties resource="db.properties">
    <property name="username" value="root"/>
    <property name="password" value="neu406"/>
</properties>
```

与上述配置properties的方式是相同的。并且经过测试，xml里面配置的属性优先级要高。、

### typeAliases

##### 作用

可以为Java类型设置一个缩写名字，用来降低一直写完全限定名的冗余。

##### 使用

给实体类起别名

```xml
<typeAliases>
    <typeAlias type="com.wyb.mybatis.poji.User" alias="User"/>
</typeAliases>
```

使用的位置：

可以使用在配置查询语句的时候：

```xml
原来：<select id="getUserList" resultType="com.wyb.mybatis.poji.User"  >
    select * from user
</select>
```

```xml
别名优化之后：<select id="getUserList" resultType="User"  >
    select * from user
</select>
```

另外的取别名的方式：也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean，会扫描实体类的包，它的默认别名为这个类的类名的小写（没有注解配置）,官方建议小写，大写也可以的。

```xml
In XML
	<typeAliases>
        <package name="com.wyb.mybatis.poji"/>
    </typeAliases>
In Mapper
    <select id="getUserList" resultType="user"  >
        select * from user
    </select>
```

如果是扫描包的情况下，有注解的话就是注解的值了。

```java
@Alias("User")
public class User {
    ...
}
```

#### Mappers

用来告诉MyBatis去哪里寻找映射的SQL语句。但是指定```xml```在哪个位置的话，配置的方式就比较多样了。

##### 使用相对类路径的资源引用

```xml
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>
```

##### 使用完全限定资源定位符

```xml
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>
```

##### 使用映射器接口实现类的完全限定类

```xml
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
```

以上的配置会告诉MyBatis去哪里寻找映射文件。