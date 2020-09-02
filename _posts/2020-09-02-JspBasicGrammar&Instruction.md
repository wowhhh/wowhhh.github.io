---
layout: post
title:  "Jsp基础语法和指令"
tags:   Jsp 
date:   2020-09-02
categories: [Java Web]

---



- 一个Jsp页面是由什么组成的？
  - Html、Js、css
  - 指令 <%@ .. %>
  - 表达式 <%= ... %>
  - 脚本片段 <% ... %>
  - 声明 <%! ... %>
  - Jsp标签 <jsp:include page="Filename" >
  - 注释 <%-- ... --%>

### 语法及其使用

#### 1：Jsp表达式

```jsp
<%--    JSP 表达式
作用：用来将程序的结果输出到客户端
<%= 表达式/变量 %>
--%>
<%= new java.util.Date()%>


在功能上相当于
<% out.println(new java.util.Date();)
```

#### 2：Jsp脚本片段

```jsp
<%--    JSP 脚本片段 --%>
    <%
        int sum = 0;
        for (int i = 0; i <= 100 ; i++)
        {
            sum+=i;
        }
        out.print("<h3>Sum="+sum+"<h3>");
    %>
编写java 代码，最后的class文件，这部分会作为Java代码出现在JspService中。
```

#### 3：在代码中嵌入Html元素

```jsp
<%--代码中嵌入Html--%>
<%
    for (int i = 0; i < 10; i++) {

%>
    <h3>Hello 嵌入 Test <%=i%></h3>
<%

    }
%>

括号可以分开，这其实比较容易理解的，因为一个Jsp中会有多个Java脚本片段，不可能都是相连的。
```

#### 4：Jsp声明

用于声明变量和方法，最后生成的class文件中，变量和方法不会在JspService方法中，会被编译到JSP生成的Java的类中。

```jsp
<%!
    static {
        System.out.printf("static");
    }
    public void test()
    {
        System.out.printf("Test");
    }
%>
```

#### 5：小总结

```jsp
<% %>
<%= %>
<%! %>
<%-- --%> 注释
```

Jsp的注释不会在Html源码中显示，不同于Html的注释。



### 指令

```
<%@ page...%>
<%@ include file=""%>
```

#### 1：page

`<%@ page errorPage="error/500.jsp" %>` in index.jsp，当index.jsp出现500错误的时候，就会跳转到500.jsp页面。

![image-20200902151439485](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-09-02/image-20200902151439485.png)

另一种配置方式（测试404）

```xml
<error-page>
    <error-code>404</error-code>
    <location>/error/404.jsp</location>
</error-page>
```

同时page中常常配置`contextType`和`pageEncoding`

#### 2：include

```jsp
<%@ include file="common/header.jsp"%>
```

通过这种方式可以把一个jsp页面包裹进来，并且在打包成class的时候，会一起打包到一个类中，那么在编写的时候就需要注意，被包裹页面里的参数或者方法可以直接使用。