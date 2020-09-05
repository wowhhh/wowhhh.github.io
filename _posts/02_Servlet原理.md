#### Servlet原理概述

Servlet是由web服务器调用，web服务器在首次收到浏览器请求之后，会生成servlet并会产生请求和响应，请求和响应会调用servlet里面的方法，Servlet类里面的service方法可以处理请求和响应，可以读取到请求的信息，可以把请求之后的响应交给response。

![image-20200828221553512](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-09-05/image-20200828221553512.png)

#### Servlet生命周期

- **Servlet实例创建**

  web容器负责加载Servlet，当web容器启动时或者是在第一次使用这个Servlet时，容器会负责创建Servlet实例，但是用户必须通过部署web.xml指定Servlet的位置，也就是Servlet所在的类名称，成功加载后，web容器会通过反射的方式对Servlet进行实例化。

- **初始化**

  在Servlet实例化之后，Servlet容器会调用init()方法，来初始化该对象，主要是为了让**Servlet对象在处理客户请求前可以完成一些初始化的工作**，例如，建立数据库的连接，获取配置信息等。对于每一个Servlet实例，**init()方法只能被调用一次**。

- **响应客户端请求**

  **service()是Servlet的核心，负责响应客户的请求**。

  每当一个客户请求一个HttpServlet对象，该对象的Service()方法就要调用，而且传递给这个方法一个“请求”（ServletRequest）对象和一个“响应”（ServletResponse）对象作为参数。

  要注意的是，在service()方法被容器调用之前，必须确保init()方法正确完成。

  容器会构造一个表示客户端请求信息的请求对象（类型为ServletRequest）和一个用于对客户端进行响应的响应对象（类型为ServletResponse）作为参数传递给service()方法。

  在service()方法中，Servlet对象通过**ServletRequest对象得到客户端的相关信息和请求信息**，在对请求进行处理后，调用**ServletResponse对象的方法设置响应信息**。

- **销毁**

  destroy()仅执行一次，在**服务器端停止且卸载Servlet**时执行该方法。

  当容器检测到一个Servlet对象应该从服务中被移除的时候，容器会调用该对象的destroy()方法，**以便让Servlet对象可以释放它所使用的资源，保存数据到持久存储设备中**，例如，将内存中的数据保存到数据库中，关闭数据库的连接等。

  当需要释放内存或者容器关闭时，容器就会调用Servlet对象的destroy()方法。

  在Servlet容器调用destroy()方法前，如果还有其他的线程正在service()方法中执行，容器会等待这些线程执行完毕或等待服务器设定的超时值到达。

  在destroy()方法调用之后，容器会释放这个Servlet对象，在随后的时间内，该对象会被Java的垃圾收集器所回收。、

#### Servlet、Servlet容器 、Tomcat

- Servlet

  Java Servlet（Java服务器小程序）是一个基于Java技术的Web组件，运行在服务器端，它**由Servlet容器所管理，用于生成动态的内容**。 **Servlet是平台独立的Java类**，编写一个Servlet，实际上就是按照Servlet规范编写一个Java类。Servlet被**编译为平台独立 的字节码**，可以被动态地加载到**支持Java技术的Web服务器**中运行。 

- Servlet 容器

  Servlet容器也叫做**Servlet引擎**，是Web服务器或应用程序服务器的一部分，用于在发送的请求和响应之上提供网络服务，解码基于 MIME（类似text/html）的请求，格式化基于MIME的响应。**Servlet没有main方法，不能独立运行，它必须被部署到Servlet容器中**，由**容器来实例化和调用 Servlet的方法**（如doGet()和doPost()），Servlet容器在Servlet的生命周期内包容和管理Servlet。在JSP技术 推出后，管理和运行Servlet/JSP的容器也称为Web容器。

  有了servlet之后，用户通过单击某个链接或者直接在浏览器的地址栏中**输入URL来访问Servlet**，Web服务器接收到该请求后，并不是将 请求直接交给Servlet，而是交给**Servlet容器**。Servlet容器实例化Servlet，调用**Servlet**的一个特定方法对请求进行处理， 并产生一个响应。这个响应由**Servlet容器返回给Web服务器**，Web服务器包装这个响应，以HTTP响应的形式发送给**Web浏览器**。

- Tomcat

  Tomcat是一个免费的开放源代码的Servlet容器

#### 工作流程

1：客户端向Tomcat发出Http请求

2：Servlet容器接收到客户端的请求

3：Servlet容器创建一个HttpRequest对象，将客户端的请求信息封装到这个对象中

4：Servlet容器创建一个HttpResponse对象

5：Servlet容器调用HttpServlet对象的Service方法，把HTTP Request对象与HttpResponse对象作为参数传给HttpServlet对象

6：HttpServlet调用HttpRequest对象的有关方法，获取Http请求信息

7：HttpServlet调用HttpResponse对象的有关方法，生成响应信息

8：Servlet容器把HttpServlet的响应结果传给客户端

#### HttpServlet

HttpServlet 指能够处理 HTTP 请求的 servlet，它在原有 Servlet 接口上添加了一些与 HTTP 协议处理方法。

HttpServlet 在实现 Servlet 接口时，覆写了 service 方法，该方法体内的代码会自动判断用户的请求方式，如为 GET 请求，则调用 HttpServlet 的 doGet 方法，如为 Post 请求，则调用 doPost 方法。所以我们在开发过程中就需要重写doGet以及doPost等方法。

#### Servlet mappings

mappings是用来干嘛的？

从上面的分析可以看到：用户通过URL访问Servlet，先进入的是Servlet容器，这个容器来帮我们定位目的的Servlet，那它是怎么知道要寻找哪个Servlet呢，这就用到了映射机制。

1：一个Servlet可以指定一个映射路径

```xml
<servlet-mapping>
	<servlet-name>name</servlet-name>
	<url-pattern>/hello</url-pattern>
</servlet-mapping>
```

2：一个Servlet可以指定多个映射路径

```xml
<servlet-mapping>
	<servlet-name>name</servlet-name>
	<url-pattern>/hello1</url-pattern>
</servlet-mapping>
<servlet-mapping>
	<servlet-name>name</servlet-name>
	<url-pattern>/hello2</url-pattern>
</servlet-mapping>
<servlet-mapping>
	<servlet-name>name</servlet-name>
	<url-pattern>/hello3</url-pattern>
</servlet-mapping>
```

3：一个Servlet可以指定通用映射路径

```xml
<servlet-mapping>
	<servlet-name>name</servlet-name>
	<url-pattern>/hello/*</url-pattern>
</servlet-mapping>
```

4：默认请求路径

```xml
<servlet-mapping>
	<servlet-name>name</servlet-name>
	<url-pattern>/*</url-pattern>
</servlet-mapping>
```

5：指定一些后缀或者前缀等待

```xml
<servlet-mapping>
	<servlet-name>name</servlet-name>
	<url-pattern>*.wyb</url-pattern> <!--*前面不能加任何映射的路径/-->
</servlet-mapping>
```
