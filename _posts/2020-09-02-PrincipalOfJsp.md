---
layout: post
title:  "Jsp前言之运行原理"
tags:   Jsp Tomcat
date:   2020-09-02
categories: [Java Web]

---



### 什么是Jsp

Java Server Pages，与Servlet一样，都是用于开发动态Web，只不过Jsp可以在页面中定义html标签以及java代码。

### Jsp与Servlet有什么关系

如果完成如下需求：通过Servlet和Jsp输出如下内容：

```html
<html>
    <body>
        <h2>
            Hello
        </h2>
    </body>
</html>
```

Servlet中要通过out对象向页面输出:

```java
out.write("<html>")
out.write("<body>")
out.write("<h2>")
```

所以这样写就很烦，但是写Jsp就像写Html，在里面使用Jsp提供的语法嵌入Java代码即可。

### Jsp与Html区别

- Html只提供静态
- Jsp页面可以嵌入Java代码，提供动态支持



### Jsp原理

#### Jsp运行过程

1：客户端请求Jsp文件，Tomcat将会根据请求的Jsp文件生成Java文件

2：Java文件再生成对应的字节码class文件

3：web容器加载class字节码

4：web容器建立相应Jsp的实例

5：调用JspInit完成初始化

6：调用Jspserver响应用户请求

7：调用JspDestory销毁创建的实例



#### 下面是从源码角度的一些发现

![image-20200902105609835](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/image-20200902105609835.png)

从上面图片看到，生成的JSP还是一样的，所以就要看服务器（Tomcat）是怎么做的了。

在Tomcat中有一个work目录，IDEA中使用Tomcat也会生成同样的Work目录。

C:\Users\17921\AppData\Local\JetBrains\IntelliJIdea2020.2\tomcat\Unnamed_JspLearning_2\work\Catalina\localhost\JspLearning_war_exploded\org\apache\jsp

可以在上述目录发现，jsp被转化成java类了

![image-20200902125415386](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-09-02image-20200902125415386.png)

在类的JspService中有许多的out.write方法用于输出html

![image-20200902125447599](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-09-02image-20200902125447599.png)

可见Jsp本质上就是Servlet，因为它所继承的东西也是继承自Servlet

```java
    public void _jspInit() { //初始化
    }

    public void _jspDestroy() { //销毁
    }

    public void _jspService(HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException { //Jsp Service
	
```

#### 从源码中发现Jsp作用

- 判断请求：

![image-20200902130323684](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-09-02/image-20200902130323684.png)

- 内置对象（供后续直接调用）

  ```
  PageContext
  Session
  ServletContext applicationContext
  ServletConfig
  JspWriter Out
  page
  HttpServletRequest
  HttpServletResponse
  ```

- 输出页面前增加的代码

  ```java
  response.setContentType("text/html; charset=UTF-8"); 设置响应页面类型
              PageContext pageContext = _jspxFactory.getPageContext(this, request, response, (String)null, false, 8192, true);
              _jspx_page_context = pageContext;
              pageContext.getServletContext();
              pageContext.getServletConfig();
              out = pageContext.getOut();
  ```

  以上的这些对象可以在Jsp中直接使用



Jsp编译后的class在访问到相关servlet的时候才生成。

Jsp页面中的java代码，就会原封不动的输出，Html代码会被out.write()写出去。