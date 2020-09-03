---
layout: post
title:  "从页面跳转看请求转发与重定向"
tags:   Jsp Request Response Redirect
date:   2020-09-03 14:15:00 +0800
categories: [Java Web]
---

### 从使用场景看页面如何完成跳转

- Servlet > Jsp

```java
resp.sendRedirect("user.jsp");
req.getRequestDispatcher("/user.jsp").forward(req,resp);
```

- Servlet > Servlet

```java
req.getRequestDispatcher("/配置在web.xml中的url-pattern").forward(req,resp);
resp.sendRedirect("配置在web.xml中的url-pattern");
```

- Jsp > Servlet/Jsp

```jsp
<jsp:forward page="/配置在web.xml中的url-pattern"></jsp:forward>
response.sendRedirect()
response.setHeader("Location","")
```

#### 请求转发与重定向

https://www.cnblogs.com/baikaizhuliangshui/p/11496377.html