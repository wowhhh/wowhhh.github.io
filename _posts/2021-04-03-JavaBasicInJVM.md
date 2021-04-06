---
layout: post
title:  "Java Basic In JVM"
tags:   Java 
date:   2021-04-05 9:00:00 +0800
categories: [JVM]


---

#### 字符串与JVM

https://blog.csdn.net/TofuCai/article/details/107505171

会创建几个对象的问题：https://blog.csdn.net/wo541075754/article/details/108212654

https://juejin.cn/post/6844904129752465416

字符串常量池相当于给字符串开辟一个常量池空间类似于缓存区，对于直接赋值的字符串（String s="xxx"）来说，在每次创建字符串时优先使用已经存在字符串常量池的字符串，如果字符串常量池没有相关的字符串，会先在字符串常量池中创建该字符串，然后将引用地址返回变量。

字符串常量池在JDK 1.7之后就移动到Java堆上面了。

JVM为了提高和减少内存开销引入了字符串常量池的概念

- String StringBuilder

- String a = "abc"

  >String a = "abc"
  >
  >String a = new String("abc");
  >
  >创建了几个对象

  >String a = new String("abc");
  >
  >创建了几个对象

  > *String* str = "abc" + "def";
  >
  >创建了几个对象

#### 自增自减与JVM

[参考：jvm，自增自减原理](https://blog.csdn.net/mazhen1991/article/details/75093077)

```java
public class JavaTest{

    public static void main(String[] args){
        int a=1,b=1,c=1,d=1;
        a++;
        ++b;
        c = c++;
        d=++d;
        System.out.println(a+"..."+b+"..."+c+"..."+d);
    }
}
```

iconst_1 : 将值为1的int入栈

istore_1：弹出操作数栈栈顶元素，保存到局部变量表第1个位置

iload_1：加载局部变量表的第1个变量到操作数栈顶

#### return与JVM

- return

  [参考](https://blog.csdn.net/romantic_jie/article/details/100065632)

  这里得联系finally才更有意思

  

#### 装箱拆箱与JVM

属于语法糖，对于int和Integer的转换，装箱得时候帮我们调用了valueOf(int)方法，拆箱得时候帮我们调用了intValue。

注意-128 到127得缓存

注意：float和double是没有缓存的，浮点类型，==没办法判相等

https://blog.csdn.net/TofuCai/article/details/107697114

[Byte、Short、Boolean、Char、Long的缓存](https://blog.csdn.net/cdecde111/article/details/60583372)

#### 重载、重写与JVM

**虚拟机如何确定正确的目标方法**

[分派](https://tobiaslee.top/2017/02/14/Override-and-Overload/)

- 静态分派：重载，调用谁在编译的时候都确定了

  **虚拟机（准确说是编译器）是通过参数静态类型作为重载的判定依据**,两个变量实际类型不同，然并卵，静态类型相同，就决定了他们会使用同一个重载函数。

- 动态分派：重写

  invokevirtual指令的多态查找

#### 反射与JVM

https://blog.csdn.net/riemann_/article/details/104033656

反射呢是 Java 语言中一个相当重要的特性，它允许正在运行的 Java 程序观测，甚至是修改程序的动态行为。表现为两点，一是对于任意一个类，都能知道这个类的所有属性和方法，二是对于任意一个对象，都能调用它的任意属性和方法。

- 反射涉及的API

  反射的使用还是比较简单的，涉及的 API 分为三类，Class、Member（Filed、Method、Constructor）、Array and Enumerated。

- 反射为什么影响性能

  在此之前，我先讲讲 JVM 是如何实现反射的。

  我们可以直接 new Exception 来查看方法调用的栈轨迹，在调用 Method.invoke() 时，是去调用 DelegatingMethodAccessorImpl 的 invoke，它的实际调用的是 NativeMethodAccessorImpl 的 invoke 方法。前者称为委派实现，后者称为本地实现。既然委派实现的具体实现是一个本地实现，那么为啥还需要委派实现这个中间层呢？其实，Java 反射调用机制还设立了另一种动态生成字节码的实现，成为动态实现，直接使用 invoke 指令来调用目标方法。之所以采用委派实现，是在本地实现和动态实现直接做切换。依据注释信息，动态实现比本地实现相比，其运行效率要快上 20 倍。这是因为动态实现无需经过 Java 到 C++ 再到 Java 的切换，但由于生产字节码比较耗时，仅调用一次的话，反而是本地实现要快上三四倍。考虑到很多反射调用仅会执行一次，JVM 设置了阈值 15，在 15 之下使用本地实现，高于 15 时便开始动态生成字节码采用动态实现。这也被称为 Inflation 机制。

  在反手说一下反射的性能开销在哪呢？平时我们会调用 Class.forName、Class.getMethod、以及 Method.invoke 这三个操作。

  - Class.forName

    在JDK的源码实现中，可以发现最终调用的是native方法forName0()，它在JVM中调用的实际是findClassFromClassLoader()，原理与ClassLoader的流程一样

  - Class.getMethod 

    会遍历该类的公有方法，如果没有匹配到，它还将遍历父类的公有方法，可想而知，这两个操作都非常耗时。
  
  - Method.invoke
  
    invoke 方法的参数是一个可变长参数，也就是构建一个 Object 数组存参数，这也同时带来了基本数据类型的装箱操作，在 invoke 内部会进行运行时权限检查，这也是一个损耗点。普通方法调用可能有一系列优化手段，比如方法内联、逃逸分析，而这又是反射调用所不能做的，性能差距再一次被放大。
  
  优化反射调用，可以尽量避免反射调用虚方法、关闭运行时权限检查、可能需要增大基本数据类型对应的包装类缓存、如果调用次数可知可以关闭 Inflation 机制，以及增加内联缓存记录的类型数目。

#### 多态与JVM

https://segmentfault.com/a/1190000021936858

分派

#### 泛型与JVM

语法糖，castcheck

https://juejin.cn/post/6844904165391466504

#### 注解与JVM

主要就是要知道注解信息是存放在哪的？在 Java 字节码中呢是通过 RuntimeInvisibleAnnotations 结构来存储的，它是一个 Annotations 数组，毕竟类、方法、属性是可以加多个注解的嘛。在数组中的每一个元素又是一个 ElementValuePair 数组，这个里面存储的就是注解的参数信息。

运行时注解可以通过反射去拿这些信息，编译时注解可通过 APT 去拿，基本上就没啥东西了。

#### 异常与JVM

https://blog.csdn.net/TofuCai/article/details/107693706

在 Java 中，所有的异常都是 Throwable 类或其子类，它有两大子类 **Error 和 Exception**。 当程序触发 Error 时，它的执行状态已经无法恢复，需要终止线程或者终止虚拟机，常见的比如内存溢出、堆栈溢出等；Exception 又分为两类，一类是受检异常，比如 IOException，一类是运行时异常 RuntimeException，比如空指针、数组越界等。

接下来我会从三个方面阐述这个问题。

首先是，**异常实例的构造**十分昂贵。这是由于在构造异常实例时，JVM 需要生成该异常的栈轨迹，该操作逐一访问当前线程的 Java 栈桢，并且记录下各种调试信息，包括栈桢所指向方法的名字、方法所在的类名以及方法在源代码中的位置等信息。

其次是，JVM 捕获异常需要异常表。每个方法都有一个异常表，异常表中的每一个条目都代表一个异常处理器，并且由 from、to、target 指针及其异常类型所构成。**form-to 其实就是 try 块**，而 **target 就是 catch 的起始位置**。当程序触发异常时，JVM 会检测触发异常的字节码的索引值落到哪个异常表的 from-to 范围内，然后再判断异常类型是否匹配，匹配就开始执行 target 处字节码处理该异常。

最后是 **finally代码块的编译**。我们知道 finally 代码块一定会运行的（除非虚拟机退出了）。那么它是如何实现的呢？其实是一个比较笨的办法，当前 JVM 的做法是，复制 finally 代码块的内容，分别放在所有可能的执行路径的出口中。