---
layout: post
title:  "单例模式 :)"
tags:   创建型模式
date:   2019-06-04 00:00:00 +0800
categories: [设计模式]

---

## 简单介绍

单例模式属于**创建型模式**，它是确保一个类最多只有一个实例，并提供一个全局访问点。也就是整个系统中只需要拥有这一个全局对象。

保证在整个软件系统中，对某个类只能存在一个对象实例，并且该类只提供一个取得其对象实例的方法。

![image-20201111165306046](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-10-19/image-20201111165306046.png)

## 应用场景

1：网站的计数器

2：Web应用的配置对象的读取，因为配置文件是共享的资源

3：数据库的连接池

4：需要设置全局的一些属性

## 优缺点

优点：

> 1：确保所有对象都访问一个实例
>
> 2：由于在系统内存中只存在一个对象，因此可以 节约系统资源
>
> 3：避免了对共享资源的多重占用

缺点

>1：不适用于需要变化的对象
>
>2：单例模式没有抽象层，不易于拓展
>
>3：单例类的职责过重，违背了“单一职责原则”

## 构建要素

1：私有化构造方法

2：私有静态实例引用

3：返回静态实例的静态公有方法

## 如何实现

> 单例模式主要有：饿汉式单例、懒汉式单例（线程不安全型、线程安全型、双重检查锁类型、静态内部类类型）、注册式（登记式）单例（枚举式单例、容器式单例）、`ThreadLocal`线程单例

### 饿汉

####  静态常量

步骤：

1：构造器私有化（防止new）

2：类的内部创建对象

3：提供一个公有的静态方法，返回实例对象

代码：

```java
//饿汉式 （静态变量）
class Singleton{
    //私有化静态方法
    private Singleton()
    {

    }
    //类的内部创建对象
    private final static Singleton singleton = new Singleton();

    //给外部提供一个获取对象的方法
    public static Singleton getInstance()
    {
        return singleton;
    }
}
```

 优缺点：

1：写法简单，在类装载的时候就完成实例化，避免了线程同步问题

2：在类装载的时候就完成实例化，没有达到懒加载的效果，如果从始至终没有使用过这个实例，就会导致内存的浪费。

#### 静态代码块

```java
//饿汉式 （静态代码块）
class Singleton{
    //私有化静态方法
    private Singleton()
    {

    }
    //类的内部创建对象
    private  static Singleton singleton ;
    static {
        singleton = new Singleton();
    }
    //给外部提供一个获取对象的方法
    public static Singleton getInstance()
    {
        return singleton;
    }
}
```

这种方法与上面的方法对比：只是将类实例化 的过程放在了静态代码块中，与上述只是写法上的区别，优缺点相同。

类加载的时候就立即初始化

创建单例对象

线程安全，因为在线程还没出现的时候就是实例化

例如：Spring中IOC容器的ApplicationContext就是饿汉式单例

- 优缺点
- 写法
- 使用场景

### 懒汉

#### 线程不安全

提供一个静态公有方法，当使用到该方法的时候，才去创建对象

```java
class Singleton{
    //私有化构造方法
    private Singleton()
    {
    }

    private static Singleton singleton;
    public static Singleton getInstance()
    {
        //当该实例为null的时候，创建并返回
        if(singleton == null)
        {
            return new Singleton();
        }
        return singleton;
    }
}
```

#### 线程安全、同步写法

```java
class Singleton{
    private Singleton()
    {
    }

    private static Singleton singleton;
    //加入同步处理方法，解决线程安全问题
    public static synchronized   Singleton getInstance()
    {
        if(singleton == null)
        {
            return new Singleton();
        }
        return singleton;
    }
}
```

解决了线程不安全问题

效率低，每个线程在想获得类的实例的时候，执行getInstance()方法都要进行同步，而其实这个方法只执行一次实例化的代码就足够了，后面的想获得该来实例，直接return就好了，方法进行同步效率太低。

不推荐使用这种方法。

#### 同步代码块（弊端很大）

先看写法，再看缺点：

```java
class Singleton {
    private Singleton() {
    }

    private static Singleton singleton;

    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                return new Singleton();
            }
        }
        return singleton;
    }
}
```

这种方法的本意是想对上面方法进行效率上面提升。但是这种同步并不能起到线程同步的作用，因为如果有另外的线程也进入了if的true分支，未来还是会往下执行，所以还是会产生多个实例

开发中也不推荐。

### 双重检查- 推荐使用

能解决线程安全问题以及效率问题

```java
class Singleton{
    private static volatile Singleton singleton;
    private Singleton()
    {

    }

    public static Singleton getInstance()
    {
        if(singleton == null)
        {
            synchronized (Singleton.class)
            {
                if(singleton == null)
                {
                    return new Singleton();
                }
            }
        }
        return singleton;
    }

}
```

### 静态内部类 - 推荐使用

```java
class Singleton{
    private Singleton()
    {
        
    }
    //静态内部类，该类中有一个静态属性Singleton
    private static class SingletonInstance
    {
        private static final Singleton INSTANCE = new Singleton();
    }
    //获取上述静态类中的属性
    public static Singleton getInstance()
    {
        return SingletonInstance.INSTANCE;
    }
}
```

1：这种方法采用了类装载机制来保证初始化实例时候只有一个线程

2：静态内部类方式在Singleton类被装载的时候并不会立即实例化，而是在需要实例化的时候，调用getInstance方法，才会装载SingletonInstance类，从而完成Singleton的实例化。

3：类的静态属性只会在第一次加载类的时候初始化，所以在这里，JVM帮助我们保证了线程的安全性，在类初始化的时候，别的线程无法进入。

### 枚举方式

```java
//使用枚举
enum Singleton{
    INSTACE;

    @Override
    public String toString() {
        return super.toString();
    }
}
```



双重检查，静态内部类，枚举，饿汉式（单线程的时候）都建议使用

## 源码分析

```java.lang.Runtime```

```java
public class Runtime {
    private static final Runtime currentRuntime = new Runtime();

    /**
     * Returns the runtime object associated with the current Java application.
     * Most of the methods of class {@code Runtime} are instance
     * methods and must be invoked with respect to the current runtime object.
     *
     * @return  the {@code Runtime} object associated with the current
     *          Java application.
     */
    public static Runtime getRuntime() {
        return currentRuntime;
    }

    /** Don't let anyone else instantiate this class */
    private Runtime() {}
    .....
}
```

## 可参考文章

[设计模式 - 单例模式（详解）看看和你理解的是否一样？](https://segmentfault.com/a/1190000020608216)

[设计模式之单例模式详解](https://juejin.im/post/6844903727262859277)

[趣谈java单例模式](https://blog.csdn.net/qsbbl/article/details/93378129)

[由Spring框架中的单例模式想到的](https://www.cnblogs.com/chengxuyuanzhilu/p/6404991.html)