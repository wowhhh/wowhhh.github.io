---
layout: post
title:  "简单工厂:)"
tags:   创建型模式
date:   2019-06-05 00:00:00 +0800
categories: [设计模式]


---

## 简单介绍

简单工厂模式属于创建型模式。可以根据参数的不同返回不同类的实例，由一个工厂对象决定创建出哪一种产品类的实例。

简单工厂模式专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类。

定义了一个创建对象的类，由这个类来封装实例化对象的行为。

## 应用场景

当开发中，当会用到大量创建某种或者某类对象的时候，就会使用到工厂模式。

- 工厂类负责创建的对象比较少：由于创建的对象较少，不会造成工厂方法中的业务逻辑太过复杂。
- 客户端只知道传入工厂类的参数，对于如何创建对象不关心：客户端既不需要关心创建细节，甚至连类名都不需要记住，只需要知道类型所对应的参数。

## 优缺点

- 将创建实例的工作与使用实例的工作分开，使用者不必关心类对象如何创建，实现了解耦；
- 把初始化实例时的工作放到工厂里进行，使代码更容易维护。 更符合面向对象的原则 & 面向接口编程，而不是面向实现编程。



- 工厂类集中了所有实例（产品）的创建逻辑，一旦这个工厂不能正常工作，整个系统都会受到影响；
- 违背“开放 - 关闭原则”，一旦添加新产品就不得不修改工厂类的逻辑，这样就会造成工厂逻辑过于复杂。

## 构建要素

|         组成(角色)         |                关系                |                     作用                     |
| :------------------------: | :--------------------------------: | :------------------------------------------: |
|     抽象产品(Product)      |           具体产品的父类           |              提供产品的公共接口              |
| 具体产品(Concrete Product) | 抽象产品的子类；工厂类创建的目标类 |              描述生产的具体产品              |
|       工厂(Creator)        |             被外界调用             | 根据传入不同参数从而创建不同具体产品类的实例 |



## 如何实现

- 创建抽象产品类 & 定义具体产品的**公共接口**
- 创建具体产品类（继承抽象产品类） & 定义生成的具体产品
- 创建工厂类，通过创建静态方法根据传入不同参数从而创建不同具体产品类的实例
- 外界调用工厂类的静态方法，传入不同参数从而创建不同具体产品类的实例

例子：

>假设目前有一个做烩面的项目：要便于烩面种类的扩展，需要达到方便维护的目的
>
>1：烩面做法有许多：羊肉烩面、牛肉烩面
>
>2：制作过程有：和面、发面、拉面、**熬汤（做法不同）**、煮面、出锅
>
>3：完整烩面店铺的销售功能

1：创建抽象产品类 & 定义具体产品的**公共接口**

```java
public abstract class StewedNoodles {
    public void he() {
        System.out.println("和面");
    }

    public void fa() {
        System.out.println("发面");
    }

    public void la() {
        System.out.println("拉面");
    }

    public abstract void ao();

    public void zhu() {
        System.out.println("煮面");
    }

    public void chu() {
        System.out.println("出锅");
    }
}
```

2：创建具体产品类（继承抽象产品类） & 定义生成的具体产品

```java
public class BeefNoodles extends StewedNoodles{

    @Override
    public void ao() {
        System.out.println("熬牛肉汤");
    }
}

public class LambNoodles extends StewedNoodles{

    @Override
    public void ao() {
        System.out.println("熬羊肉汤");
    }
}
```



3：创建工厂类，通过创建静态方法根据传入不同参数从而创建不同具体产品类的实例

```java
public class NoodlesFactory {
    public static StewedNoodles Manufacture(String noodlesName) {
        switch (noodlesName) {
            case "Beef":
                return new BeefNoodles();
            case "Lamb":
                return new LambNoodles();
            default:
                return null;
        }
    }
}
```



4：外界调用工厂类的静态方法，传入不同参数从而创建不同具体产品类的实例

```java
public class SimpleFactoryPatternTest {
    public static void main(String[] args) {
        NoodlesFactory noodlesFactory = new NoodlesFactory();
        noodlesFactory.Manufacture("Beef").chu();
        noodlesFactory.Manufacture("Lamb").chu();
    }
}
```



## 源码分析

method : ```createCalendar()```in ```java.util.Calendar```

```java
    private static Calendar createCalendar(TimeZone zone,
                                           Locale aLocale)
    {
        ...
        //根据caltype不同，创建不同的对象
        if (aLocale.hasExtensions()) {
            String caltype = aLocale.getUnicodeLocaleType("ca");
            if (caltype != null) {
                switch (caltype) {
                case "buddhist":
                cal = new BuddhistCalendar(zone, aLocale);
                    break;
                case "japanese":
                    cal = new JapaneseImperialCalendar(zone, aLocale);
                    break;
                case "gregory":
                    cal = new GregorianCalendar(zone, aLocale);
                    break;
                }
            }
        }
        ...
    }
```



## 参考文章

[简单工厂模式（SimpleFactoryPattern）- 最易懂的设计模式解析](https://blog.csdn.net/carson_ho/article/details/52223153)