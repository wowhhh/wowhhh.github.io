---
layout: post
title:  "Jsp编译后存放的位置"
tags:   Jsp Idea
date:   2020-09-02
categories: [Java Web]
---

本文记录部署Tomcat之后，运行完相关web项目以及相关Jsp在编译完成，Jsp在本地所处的位置。

- 编译器的workpace

  [参考文章](https://blog.csdn.net/qq_31908651/article/details/81938048)中提到的位置为`C:/Users/登录名/.IntelliJIdea2017.2/system/tomcat/Tomcat-pure_工程名/work/Catalina/localhost/appcontext名称/org/apache/jsp`

  本人本机实际位置：`C:/Users/登录名/AppData/Local/JetBrains/IntelliJIdea2020.2/tomcat/Tomcat-pure_工程名/work/Catalina/localhost/appcontext名称/org/apache/jsp`

  其中`JetBrains/IntelliJIdea2020.2`为不同之处。

- Tomcat安装目录

  `Tomcat安装目录\work\Catalina\localhost\ROOT\org\apache\jsp`



