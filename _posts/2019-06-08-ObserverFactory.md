---
layout: post
title:  "观察者模式 :)"
tags:   行为型模式
date:   2019-06-08 10:00:00 +0800
categories: [设计模式]
---

## 简单介绍

观察者模式是一种对象行为型模式。它表示的是一种对象与对象之间具有依赖关系，当一个对象发生改变的时候，这个对象所依赖的对象也会做出反应。

Spring 事件驱动模型就是观察者模式很经典的一个应用。Spring 事件驱动模型非常有用，在很多场景都可以解耦我们的代码。

比如我们每次添加商品的时候都需要重新更新商品索引，这个时候就可以利用观察者模式来解决这个问题。

也就是说，对象之间存在着依赖关系，当一个对象的状态发生改变，依赖此对象的对象就都会收到通知，并自动更新。

举个栗子：

A对象为手机处理器供货商；B、C、D依赖此供货商进行批量生产手机，但是A此时处于缺货状态；当A的处理器有货的时候，此时A的状态就发生了改变，通知B、C、D，三者做出各自对应的决策。

[此文举的例子不辍：](https://juejin.cn/post/6844904100459446285)

>微信公众号，如果一个用户订阅了某个公众号，那么便会收到公众号发来的消息，那么，公众号就是『被观察者』，而用户就是『观察者』
>
>气象站可以将每天预测到的温度、湿度、气压等以公告的形式发布给各种第三方网站，如果天气数据有更新，要能够实时的通知给第三方，这里的气象局就是『被观察者』，第三方网站就是『观察者』
>
>MVC 模式中的模型与视图的关系也属于观察与被观察

## 优缺点

### 优点

- 降低了目标与观察者之间的耦合关系，两者之间是抽象耦合关系
- 目标与观察者之间建立了一套触发机制
- 支持广播通信
- 符合“开闭原则”的要求

### 缺点

- 目标与观察者之间的依赖关系并没有完全解除，而且有可能出现循环引用’
- 当观察者对象很多的时候，通知的发布会花费很多时间，影响程序的效率

## 构建要素

|           组成(角色)           |                       关系                       | 作用                                                         |
| :----------------------------: | :----------------------------------------------: | ------------------------------------------------------------ |
|        目标（Subjects）        | 又称抽象被观察者，一般使用抽象类或者接口进行实现 | 它是指被观察的对象。                                         |
|  具体目标（ConcreteSubject）   |                   目标类的子类                   | 通常它包含经常发生改变的数据，当它的状态发生改变时，向它的各个观察者发出通知。 |
|        观察者(Observer)        |            又称抽象观察者，一般是接口            | 观察者将对观察目标的改变做出反应。                           |
| 具体观察者（ConcreteObserver） |                    实现观察者                    | 在具体观察者中维护一个指向具体目标对象的引用，它存储具体观察者的有关状态，这些状态需要和具体目标的状态保持一致 |



## 如何实现

- 1：抽象被观察者;

  被观察者是被其他对象观察的，当被观察者的状态发生改变，其他对象应该能立刻接收到。

  这里把被观察者抽象成接口；并且需要具备**添加观察者、删除观察者以及通知观察者**的功能！

- 2：定义观察者接口

  用于定义被观察者状态发生改变时候，自己需要做出的反应。

- 3：定义具体目标(实现被观察者)

  实现被观察者接口或者继承被观察者类。实现具体的方法，即状态如何发生改变。

- 4：定义观察者(实现观察者接口)

  用于实现各个观察者在接收到被观察者对象之后做出的反应。

## 例子

我们就以手机处理器供货商和各大手机厂商为例：

- 1：抽象被观察者

  此时抽象供货商的行为

  ```java
  public abstract class Supplier {
      private Vector<Manufacturer> manufacturers = new Vector<>();
      public void addSupplier(Manufacturer manufacturer)
      {
          manufacturers.add(manufacturer);
      }
      public void deleteSupplier(Manufacturer manufacturer)
      {
          manufacturers.remove(manufacturer);
      }
      public void notifySupplier()
      {
          for(Manufacturer manufacturer:manufacturers)
          {
              manufacturer.makePhone();
          }
      }
      //供货商状态发生改变的方法
      abstract void newCPU();
  
  }
  ```

- 2：抽象观察者

  手机厂商的共同行为

  ```java
  public interface Manufacturer {
      public void makePhone();
  }
  ```

- 3:定义具体目标

  ```java
  public class QualcommSupplier extends Supplier{
      @Override
      void newCPU() {
          System.out.println("Make New CPU!");
      }
  }
  ```

- 4:定义具体观察者

  ```java
  public class XiaomiManufacturer implements Manufacturer{
      @Override
      public void makePhone() {
          System.out.println("Xiaomi 11 Begin!");
      }
  }
  
  public class OppoManufacturer implements Manufacturer{
      @Override
      public void makePhone() {
          System.out.println("Reno 5 Pro Begin!");
      }
  }
  ```

- 5:主程序

  ```java
  public class Main {
      public static void main(String[] args) {
          Supplier Qualcomm = new QualcommSupplier();
          Manufacturer xiaomi = new XiaomiManufacturer();
          Manufacturer oppo = new OppoManufacturer();
          Qualcomm.newCPU();
          Qualcomm.addSupplier(xiaomi);
          Qualcomm.addSupplier(oppo);
  
          Qualcomm.notifySupplier();
          
          Qualcomm.newCPU();
          Qualcomm.deleteSupplier(oppo);
          Qualcomm.notifySupplier();
      }
  }
  ```

  

## 参考文章

[观察者模式，一篇就搞定](https://juejin.cn/post/6844904100459446285)