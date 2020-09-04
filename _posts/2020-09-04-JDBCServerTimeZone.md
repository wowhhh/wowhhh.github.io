---
layout: post
title:  "[已解决]JDBC连接出现The server time zone value '�й���׼ʱ��' is unrecognized or represents more than one time zone."
tags:   JDBC MySql Bug解决
date:   2020-09-04 10:00:00 +0800
categories: [JDBC]
---

- 问题描述

  JDBC连接Mysql数据库的时候，出现下图中的问题

![image-20200904102315305](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-09-04/image-20200904102315305.png)

观察下面提示信息发现需要配置server time zone

```verilog
Caused by: com.mysql.cj.exceptions.InvalidConnectionAttributeException: The server time zone value '�й���׼ʱ��' is unrecognized or represents more than one time zone. 
You must configure either the server or JDBC driver (via the 'serverTimezone' configuration property) to use a more specifc time zone value if you want to utilize time zone support.
```

- 解决方案

  ```java
  //修改后
  String url = "jdbc:mysql://localhost:3306/learningtest?serverTimezone=UTC";
  //修改前
  String url = "jdbc:mysql://localhost:3306/learningtest";
  ```

  

  