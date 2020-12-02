---
layout: post
title:  "hashCode与equals--转载"
tags:   Java 面试
date:   2020-12-01 10:00:10 +0800
categories: [Java]
---

https://snailclimb.gitee.io/javaguide/#/docs/java/basis/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86?id=_128-%e5%92%8cequals%e7%9a%84%e5%8c%ba%e5%88%ab

[https://www.cnblogs.com/skywang12345/p/3324958.html](java 泛型详解-绝对是对泛型方法讲解最详细的，没有之一)

两者在什么情况下有关？

> 某个类会被用于创建hash的结构的时候，比如HashSet，Hashtable, HashMap

为什么两者有关的话，重写equals就要重写hashcode

> 默认的hashcode方法，是根据对象来创建Hashcode的，所以如果我们只重写了equals方法，当两个对象的内容是相等的话，他们的hashcode值是不同的。这样就会造成这两个对象会同时存在于hash结构中，比如HashSet
>
> 这时候我们就需要重写hashcode的生成方法。
>
> 比如下面这个是根据类中的name属性来生成Hashcode，这样即使两个对象的地址不同，内容相同的话，其Hashcode也是相同的。

```java
/** 
50          * @desc重写hashCode 
51          */  
52         @Override
53         public int hashCode(){  
54             int nameHash =  name.toUpperCase().hashCode();
55             return nameHash ^ age;
56         }
```