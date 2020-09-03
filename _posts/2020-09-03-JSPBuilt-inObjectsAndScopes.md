---
layout: post
title:  "Jsp内置对象以及作用域"
tags:   Jsp 九大内置对象 四大作用域
date:   2020-09-03 09:42:00 +0800
categories: [Java Web]

---



内置对象是指不需要预先声明就可以在脚本代码和表达式中随意使用的。

### 九大内置对象的分类

- 输入输出对象：out对象、response对象、**request**对象
- 通信控制对象：**pageContext**对象、**Session**对象、**application**对象
- Servlet对象：page对象、config对象
- 错误处理对象：exception对象



### Jsp四大作用域

- **Page**范围 : 只在一个页面保持数据（PageContext）
- request范围：只在一个请求中保存数据,请求转发会携带这个数据（javax.servlet.httpServletRequest）
- **Session**范围：在一次会话中保存数据，仅供单个用户使用，从打开浏览器到关闭浏览器(javax.servlet.http.HttpSession)
- Application范围：在整个服务器中保存数据，全部用户共享，从打开服务器到关闭服务器(javax.servlet.ServletContext)



### 九大对象详解

- **PageContext**

  ```jsp
  <%--内置对象--%>
  <%
      pageContext.setAttribute("Key01","Value01");
      request.setAttribute("Key02","Value02");
      session.setAttribute("Key03","Value03");
      application.setAttribute("Key04","Value04");
  %>
  <%
  //    通过pageContext取出我们保存的值 Value01 null null null
      String name1 = (String) pageContext.getAttribute("Key01");
      System.out.printf(name1);
      String name2 = (String) pageContext.getAttribute("Key02");
      String name3 = (String) pageContext.getAttribute("Key03");
      String name4 = (String) pageContext.getAttribute("Key04");
  %>
  <%--使用EL表达式输出--%>
  <h3>${"?????"}</h3> <%--?????--%>
  <h3>${name1}</h3>
  <h3>${name2}</h3>
  <h3>${name3}</h3>
  <h3>${name4}</h3>
  <h3>${name5}</h3>
  ```

  为什么我的el表达式没输出？？？

  可能会解决的[参考链接](https://blog.csdn.net/qq_33271690/article/details/80596012?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-4.add_param_isCf&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-4.add_param_isCf)
  
  ```jsp
  <%--内置对象--%>
  <%
      pageContext.setAttribute("Key01","Value01");
      request.setAttribute("Key02","Value02");
      session.setAttribute("Key03","Value03");
      application.setAttribute("Key04","Value04");
  %>
  <%
  	//从底层到高层，
      String name1 = (String) pageContext.findAttribute("Key01");
      System.out.printf(name1);
      String name2 = (String) pageContext.findAttribute("Key02");
      String name3 = (String) pageContext.findAttribute("Key03");
      String name4 = (String) pageContext.findAttribute("Key04");
  %>
  ${name1}
  <%--使用EL表达式输出--%>
  <h3>${"?????"}</h3> <%--?????--%>
  <h3>${name1}</h3> <h3>Jsp取值：<%=name1%></h3>
  <h3>${name2}</h3> <h3>Jsp取值：<%=name2%></h3>
  <h3>${name3}</h3> <h3>Jsp取值：<%=name3%></h3>
  <h3>${name4}</h3> <h3>Jsp取值：<%=name4%></h3>
  
  Jsp取值成功输出
  ```

如果新建一个jsp，仅使用上述打印的代码，则只有name3和name4输出，这就是跟作用域有关了，pageContext仅在本页面生效，Request仅在请求转发中有效。

如果再新建一个jsp,并在上面的jsp中加入`pageContext.forward("xxx.jsp")`，则name2、name3、name4会显示。