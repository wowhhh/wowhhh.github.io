---
layout: post
title:  "Install Tomcat!"
tags:   Tomcat
date:   2019-08-31 19:42:35 +0800
categories: [Utils Servers DevelopmentTools]

---



## Tomcat

- 如果学习

  下载Tomcat ；安装 ；了解配置文件和目录结构 ；了解作用

- 目录

  安装；Tomcat目录结构；如何启动Tomcat；Tomcat配置；web程序应有的结构


### 安装（Linux）

1 : 下载链接 https://tomcat.apache.org/download-90.cgi

![image-20200825195834183](https://i.loli.net/2020/09/01/D2mPKgIeRtBT1UX.png)

2：相关命令

```
mv apache-tomcat-9.0.37.tar.gz /usr/local/tomcat/

tar -zxvf apache-tomcat-8.0.50.tar.gz

cd apache-tomcat-9.0.37/bin

./startup.sh

（访问测试）http://localhost:8080/
```

![image-20200825195958345](https://i.loli.net/2020/09/01/toNyvbf56PBwKA7.png)

### Tomcat目录结构

![image-20200825203116644](https://i.loli.net/2020/09/01/nZhlHzuxsE6tDTJ.png)

- bin : 启动和关闭Tomcat的脚本文件
- conf ： Tomcat服务器的配置文件
- lib ：Tomcat服务器的支撑jar包
- logs ：Tomcat的日志文件
- temp ：运行时产生的临时文件
- webapps ：web应用所在的目录，即供外界访问的web资源的存放目录
- work：Tomcat的工作目录

### 如何启动Tomcat

- Linux

  `./startup.sh`

- Windows

  双击`startyp.bat`

详见Tomcat中的`Running.txt`

### Tomcat的配置

Tomcat服务核心配置文件在`conf`文件夹下的`server.xml`中。此配置文件中配置了和`lcoahost、8080`有关的信息，使得`http://localhost:8080`可以访问到`webapps`目录下的`ROOT`目录中的`index.jsp`，也就是显示出来上述截图中的的页面。

- 如果将`servel.xml`中的`<Connector...>`标签中port属性从`8080`修改为`2333`，能不能从`http://localhost:2333`访问到上述页面？

  答：可以

- 如果将`servel.xml`中的`<Host...>`标签中name属性从`localhost`修改为`www.sakgds.com`，能不能从`http://www.sakgds.com:8080`访问到上述页面？

  答：不可以

  这是因为在windows中，位于`C:\windows\System32\drivers\etc`文件夹下的Host文件中，对于之前的Localhost，此文件做了一个127.0.0.1到Localhost的映射，但是www.sakgds.com并没有此映射，所以对于www.sakgds.com就会像寻找www.google.com一样去寻找此网址的IP。而不会映射到127.0.0.1这个IP地址。

  解决方案就是，在Host文件中加一个从127.0.0.1到www.sakgds.com的映射。

  其实学过计算机网络就明白，IP到网址是有一个映射关系的，对于Windows，因为Host自己配置了Localhost到本地IP的映射，所以就能找到这个map，但是www.sakgds.com既不存在于本机，也不存在于域名服务器。

### web网站应有的结构

```java
--webapps ： Tomcat服务器的web目录
	-ROOT
	-wybstudy ：网站的目录名
		- WEB-INF
			- classes : java程序
			- lib: web应用所依赖的jar包
			- web.xml : 网站配置文件
		- index.html 默认的首页
		- static
			- css
				- style.css
			- js
			- img
		- ...
```

