---
layout: post
title:  "String StringBuffer StringBuilder--转载"
tags:   Java 面试
date:   2020-12-01 10:00:10 +0800
categories: [Java]
---

https://snailclimb.gitee.io/javaguide/#/docs/java/basis/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86?id=_251-string-stringbuffer-%e5%92%8c-stringbuilder-%e7%9a%84%e5%8c%ba%e5%88%ab%e6%98%af%e4%bb%80%e4%b9%88-string-%e4%b8%ba%e4%bb%80%e4%b9%88%e6%98%af%e4%b8%8d%e5%8f%af%e5%8f%98%e7%9a%84

https://github.com/Snailclimb/JavaGuide/issues/675

https://blog.csdn.net/weixin_41101173/article/details/79677982

线程安全

性能

三者使用如何抉择

String

```java
        String x = "123";
        System.out.println(x.hashCode());
        x = "456";
        System.out.println(x.hashCode());
48690
51669
```



StringBuffer

StringBuilder