---
layout: post
title:  "适配器模式 :)"
tags:   结构型模式
date:   2019-06-09 10:00:00 +0800
categories: [设计模式]
---

## 简述

适配器模式是将一个类的接口转换成客户希望的另外一个接口，使得原本由于接口不兼容而不能一起工作的那些类能一起工作。适配器模式分为类结构型模式和对象结构型模式两种，前者类之间的耦合度比后者高，且要求程序员了解现有组件库中的相关组件的内部结构，所以应用相对较少些。

为何而出现：需要开发的具有某种业务功能的组件在现有的组件库中已经存在，但它们与当前系统的接口规范不兼容，如果重新开发这些组件成本又很高，这时用适配器模式能很好地解决这些问题。

## 优缺点

- 优点

  >1：提高复用性
  >
  >系统需要使用现有的类，而此类接口不符合系统的需要，那么通过适配器模式就可以让这些功能得到很好的复用。
  >
  >2：透明
  >
  >对客户端来说，客户端可以调用同一接口，因而对客户端来说是透明的。这样做更简单 & 更直接
  >
  >3：更容易扩展
  >
  >在实现适配器功能的时候，可以调用自己开发的功能，从而自然地扩展系统的功能。
  >
  >4：解耦性
  >
  >将目标类和适配者类解耦，通过引入一个适配器类重用现有的适配者类，而无需修改原有代码
  >
  >5：符合开闭原则
  >
  >同一个适配器可以把适配者类和它的子类都适配到目标接口；可以为不同的目标接口实现不同的适配器，而不需要修改待适配类

- 缺点

  增加系统复杂性、降低代码可读性。

## 构建要素

### 类的适配器模式

|       角色        | 关系 |                             作用                             |
| :---------------: | :--: | :----------------------------------------------------------: |
|   目标(Target)    | 接口 |          当前系统所期待的接口，可以是抽象类或者接口          |
| 适配者类(Adaptee) |  类  |            被访问，被适配的现存组件库中的组件接口            |
| 适配器类(Adapter) |  类  | 转换器，通过继承或引用适配者的对象，把适配者接口转换成目标接口，让客户按目标接口的格式访问适配者。 |

![image-20201227142719750](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-12-12/image-20201227142719750.png)

### 对象的适配器模式

异同

- 异：与类的适配器模式不同的是，对象的适配器模式不是使用继承关系连接到Adaptee类，而是使用**委派关系连接到Adaptee类**。

- 同：目的都是把需要适配的类的API转化成目标类的API

  ![image-20201227144319233](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-12-12/image-20201227144319233.png)

## 如何实现

### 类的适配器模式

- Step 1：创建Target接口，包含当前系统期待调用的方法

  ```java
  public interface Target {
      //目标系统会实例化实现了Target的类，并调用Request方法
      void Request();
  }
  ```

- Step 2：创建源角色，用于被适配的接口或者类

  ```java
  public class Adaptee {
      //需要适配器进行适配的方法
      public void otherRequest()
      {
          System.out.println("Need Adapted");
      }
  }
  ```

- Step 3：创建适配器，用于适配

  ```java
  public class Adapter extends Adaptee implements Target{
  	//这样方式的适配，系统就可以实例化Adapter进行.Request()的调用了
      @Override
      public void Request() {
          this.otherRequest();
      }
  }
  ```

- 测试

  ```java
  public class Main {
      public static void main(String[] args) {
          Target target = new Adapter();
          target.Request();
      }
  }
  ```

### 对象的适配器模式

- Step 1：创建Target接口，为系统期待调用的

  ```java
  public interface Target {
  //    系统所期待调用的方法
      void Request();
  }
  ```

  

- Step 2：创建Adaptee类，为适配器需要适配的

  ```java
  public class Adaptee {
  //    为系统所需要适配的方法
      public void oldRequest()
      {
          System.out.println("Need Adapted");
      }
  }
  ```

  

- Step 3：创建Adapter类，用于适配，这里采用对象的适配器模式

  ```java
  public class Adapter implements Target{
      Adaptee adaptee;
  
      public Adapter(Adaptee adaptee) {
          this.adaptee = adaptee;
      }
  
      @Override
      public void Request() {
          adaptee.oldRequest();
      }
  }
  ```

  

- Step 4：测试

  ```java
  public class Main {
      public static void main(String[] args) {
          Target target = new Adapter(new Adaptee());
          target.Request();
      }
  }
  ```

## 参考文章

[适配器模式](https://www.jianshu.com/p/9d0575311214)