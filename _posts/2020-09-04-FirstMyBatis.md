---
layout: post
title:  "作为小白，你需要自己动手尝试的第一个MyBatis实例"
tags:   JDBC MySQL Junit
date:   2020-09-04 21:25:00 +0800
categories: [MyBatis]

---

如果你第一次打开Mybatis提供的中文文档，势必会一头雾水（大佬除外），MyBatis的文档的确很详细，但是没有很好照顾到第一次接触MyBatis的用户。下面的内容是一步一步搭建出来一个MyBatis的示例。

#### 00 : 数据库方面

- 数据库创建

  ```sql
  CREATE DATABASE mybatis
  ```

- 表创建

  ```java
  CREATE TABLE user(
  `id` INT AUTO_INCREMENT,
  `username` VARCHAR(30) NOT NULL,
  `password` VARCHAR(30) NOT NULL,
  PRIMARY KEY(`id`)
  )ENGINE=INNODB DEFAULT CHARSET=utf8;
  ```

- 插入相关数据

  按照上述表结构插入几条数据即可。



#### 01 : Maven相关依赖的导入

如官方文档所说：

如果使用 Maven 来构建项目，则需将下面的依赖代码置于 pom.xml 文件中：

```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
```

鉴于要连接MySql以及测试，所以多加了两个依赖。

```xml
<dependencies>
    <!--For Mabatis-->
    <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.5</version>
    </dependency>
    <!--For MySql-->
    <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.21</version>
    </dependency>
    <!--For Junit-->
    <!-- https://mvnrepository.com/artifact/org.junit.jupiter/junit-jupiter-api -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.6.1</version>
        <scope>test</scope>
    </dependency>

</dependencies>
```

#### 02 : MyBatis配置文件

[官方文档](https://mybatis.org/mybatis-3/zh/getting-started.html)提到这一点是这样描述的:

> XML 配置文件中包含了对 MyBatis 系统的**核心设置**，包括获取数据库连接实例的数据源（**DataSource**）以及决定事务作用域和控制方式的事务管理器（**TransactionManager**）。
>
> ...
>
> 当然，还有很多可以在 XML 文件中配置的选项，上面的示例仅罗列了最关键的部分。 注意 **XML 头部的声明**，它用来验证 XML 文档的正确性。**environment** 元素体中包含了事务管理和连接池的配置。**mappers** 元素则包含了一组映射器（mapper），这些映射器的 XML 映射文件包含了 SQL 代码和映射定义信息。

需要关注的是基础的XML包括XML头、environment、mappers元素，并且配置了MyBatis的核心功能。

我们模仿官方的示例，根据自己的环境配置如下：

文件名：**mybatis-config.xml**

存放位置：存放在了**resources**目录下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--核心配置文件-->
<configuration>
<!--    environments可以配置多个环境，此处的是development，还可以配置test等-->
    <environments default="development">
        <environment id="development">
            <!--数据库配置-->
            <!--事务管理器-->
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
<!--                驱动-->
                <property name="driver" value="com.mysql.jdbc.Driver"/>
<!--                连接数据库的url，后缀是设置时区，适配中文字符等等-->
                <property name="url" value="jdbc:mysql://127.0.0.1:3306/mybatis?serverTimezone=UTC&amp;
useUnicode=true;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <!--这一步是在编写玩UserMapper的时候添加在这里的-->
    <mappers>
        <mapper resource="UserMapper.xml"/>
    </mappers>
</configuration>
```

#### 03 : 工具类编写

为什么编写工具类？在官方文档有这么提到

> **每个**基于 MyBatis 的应用都是以一个 ***SqlSessionFactory*** 的实例为核心的。SqlSessionFactory 的实例可以通过 ***SqlSessionFactoryBuilder*** 获得。而 SqlSessionFactoryBuilder 则可以从 **XML 配置文件**或一个预先配置的 Configuration 实例来构建出 SqlSessionFactory 实例。
>
> ...
>
> 既然有了 SqlSessionFactory，顾名思义，我们可以从中获得 **SqlSession** 的实例。

那也就是说，在使用MyBatis的时候，其实都需要创建SqlSessionFactoryBuilder、SqlSessionFactory和SQL Session，但是每次都创建未免也太复杂了，不如抽象成一个工具类。

```java
    //    初始化的时候就加载
    private static SqlSessionFactory sqlSessionFactory;
    static {
        try {
            //获取sqlSessionFactory对象
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    //从SqlSessionFactory获取SqlSession
    public static SqlSession getSqlSession()
    {
//        SqlSession session = sqlSessionFactory.openSession()
        return sqlSessionFactory.openSession();
    }
```

上述代码可能有陌生的地方就是**获取SqlSessionFactory对象**，这是官方提供的书写方式，并且整个过程符合工厂设计模式的写法。

```getSqlSession```方法就是返回SqlSession对象。

#### 04 : 实体类编写

创建与数据库对应的User类，其中User的属性名称建议和数据库中的是一模一样的，这样也会方便后续的编写。

```java
public class User {
    private int id;
    private String username;
    private String password;
    
    //Constructor
    //getter & setter
}
```

#### Mapper的编写与映射

在编写Mapper之前，需要搞懂Mapper是什么，学到这里，肯定记得以前MVC结构的时候，定义dao，并且实现dao里面的接口，我们假如对上述User类创建了UserDao，并且实现了UserDao为UserDaoImpl。

那么这里的Mapper就等同于MVC架构中的UserDao，都是提供同样功能的接口，并且在做Mapper映射的过程也相当于MVC架构中实现了UserDao。至于为什么是这样，当我们后续测试功能的时候就会明白，见如下代码：

- UserMapper

  ```java
  public interface UserMapper {
      List<User> getUserList();
  }
  ```

  ​	那我们对比一下MVC中的UsesDao是怎么编写的

  ```java
  public interface UserDao {
      List<User> getUserList();
  }
  ```

  ​	是不是差不多👀

- UserMapper.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <!--namespace指定所绑定的mapper(dao),所以namespace的内容就是与之映射的mapper的位置
  这样相当于绑定起来了，类似于以前...Impl实现了...Dao
  -->
  <mapper namespace="com.wyb.mybatis.dao.UserMapper">
      <!--
      id 对应方法名字
      -->
      <select id="getUserList" resultType="com.wyb.mybatis.poji.User">
          select * from user
      </select>
  </mapper>
  ```

  注释也提到了namespace目的就是对应到要映射的Dao，并且在mapper内部完成select、insert等等操作，我们这里完成select操作，因为Dao里面提供的接口也是做查询使用。

  看一下UserDaoImpl中要怎么写：

  ```java
  public class UserDaoImpl implements UserDao {
      @Override
      public List<User> getUserList() {
          String sql = "select * from user" ;
          //通过JDBC查询结果得到
          //users
          return users;
      }
  }
  ```

  这时候就发现namespace约等价于implements，对于查询操作，Impl里面自己要通过JDBC去访问获取，而MyBatis已经帮我们封装好了。

  那么问题来了，Mapper里面的resultType是什么的？其实这个地方有两个可以选择（resultType和resultMap），二选一：

  - resultType

    > 直接表示返回类型
    >
    > **当提供的返回类型属性是resultType时，MyBatis会将Map里面的键值对取出赋给resultType所指定的对象对应的属性。所以其实MyBatis的每一个查询映射的返回类型都是ResultMap，只是当提供的返回类型属性是resultType的时候，MyBatis对自动的给把对应的值赋给resultType所指定对象的属性。**

  - resultMap

    > 对外部 resultMap 的命名引用
    >
    > **当提供的返回类型是resultMap时，因为Map不能很好表示领域模型，就需要自己再进一步的把它转化为对应的对象，这常常在复杂查询中很有作用。**

- Mapper in mybatis-config.xml

  别忘了把创建并且映射好的Mapper添加到mybatis的config xml中，否则是找不到的。

  这里我只所以直接写了resource="UserMapper.xml"，是因为我把xml放在了resources下，如果放在了其他目录

  ```xml
  <mappers>
      <mapper resource="UserMapper.xml"/>
  </mappers>
  ```

#### 05 : 测试

上述编写完成后就可以编写测试类进行测试了。

```java
    @Test
    public void test() {
        SqlSession sqlSession = null;
        try{
            //获取SqlSession
            sqlSession = MyBatisUtils.getSqlSession();
            //执行sql 方式1 getMapper
            UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
            List<User> users = userMapper.getUserList();
            for(User user:users)
            {
                System.out.println(user.getUsername());
            }
//        方式2：sqlSession.selectList("com.wyb.mybatis.dao.UserMapper.getUserList");
        }
        catch (Exception e)
        {
            //释放
            sqlSession.close();
        }
        finally {
            //释放
            sqlSession.close();
        }
    }
```

这里需要关注官网所说的：

> 每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是**不能被共享**的，所以它的最佳的作用域是请求或方法作用域。 绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。 也绝不能将 SqlSession 实例的引用放在任何类型的托管作用域中，比如 Servlet 框架中的 HttpSession。 如果你现在正在使用一种 Web 框架，考虑将 SqlSession 放在一个和 HTTP 请求相似的作用域中。 换句话说，每次收到 HTTP 请求，就可以打开一个 SqlSession，返回一个响应后，就关闭它。 这个关闭操作很重要，**为了确保每次都能执行关闭操作，你应该把这个关闭操作放到 finally 块中**。 下面的示例就是一个确保 SqlSession 关闭的标准模式

确保SqlSession是请求或方法作用域，不能是类，更不能是静态域。

确保每次执行了释放操作，建议```try-catch```放到finally块中。

到此，运行，显示，一气呵成~

![image-20200904211052094](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-09-04/image-20200904211052094.png)