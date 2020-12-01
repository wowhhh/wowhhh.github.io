---
layout: post
title:  "抽象工厂模式 :)"
tags:   创建型模式
date:   2019-06-06 00:00:00 +0800
categories: [设计模式]



---

## 简单介绍

抽象工厂模式为一个**产品家族**提供了统一的创建接口。当需要这个产品家族的某一系列的时候，可以从抽象工厂中选出相对应的系列来创建一个具体的工厂类别。

上文提到的工厂方法模式只能创建一类产品，但是在实际过程中，一个工厂往往需要生产多类产品。

应用场景

## 优缺点

#### 优点

降低耦合

更符合开-闭原则

符合单一职责原则

不适用静态工厂方法，可以形成基于继承的等级结构

#### 缺点



## 构建要素

|         组成(角色)          |                关系                |                         作用                          |
| :-------------------------: | :--------------------------------: | :---------------------------------------------------: |
| 抽象产品族(AbstractProduct) |         **抽象产品**的父类         |                描述抽象产品的公共接口                 |
|      抽象产品(Product)      |         **具体产品**的父类         |                描述具体产品的公共接口                 |
| 具体产品(Concrete Product)  | 抽象产品的子类; 工厂类创建的目标类 |                  描述生产的具体产品                   |
|      抽象工厂(Creator)      |           具体工厂的父类           |                描述具体工厂的公共接口                 |
| 具体工厂(Concrete Creator)  |     抽象工厂的子类;被外界调用      | 描述具体工厂；实现FactoryMethod工厂方法创建产品的实例 |

## 如何实现

- 1:创建抽象工厂类，定义具体工厂的公共接口

- 2:创建抽象产品族类，定义抽象产品的公共接口

  > 比如"手机"族，就有"打电话"，"发短信"....等等共同的一些接口

- 3:创建抽象产品类，继承抽象产品族类，定义具体产品的公共接口

  > 比如"小米"、"华为"等，就有各自抽象产品里面特有的一些接口，"MIUI"、"EIUI"

- 4：创建具体产品类，继承抽象产品类，定义生产的具体产品

  > 比如小米MIX 2s、小米10 Pro

- 5：创建具体工厂类，继承抽象工厂类，定义常见对应具体产品实例的方法

- 6：客户端通过实例化具体的工厂类，并调用其创建不同目标产品的方法创建不同具体产品类的实例

## 例子

> **背景：**小成有两间塑料加工厂（A厂仅**生产容器类产品**；B厂仅**生产模具类产品**）；随着客户需求的变化，A厂所在地的客户需要也模具类产品，B厂所在地的客户也需要容器类产品；
>
> **冲突：**没有资源（资金+租位）在当地分别开设多一家注塑分厂
>
> **解决方案：**在原有的两家塑料厂里增设生产需求的功能，即A厂能生产容器+模具产品；B厂间能生产模具+容器产品。

- 1:创建抽象工厂类，定义具体工厂的公共接口

  上述例子工厂都要去实现的功能是即生产容器类产品也生产模具类产品，所以此接口里面有两个待实现的方法：

  ```java
  public abstract class Factory {
      public abstract AbstractProduct ManufactureContainer();
      public abstract AbstractProduct ManufactureMould();
  }
  ```

  

- 2:创建抽象产品族类，定义抽象产品的公共接口

  上述例子中的产品都容器类产品和模具类产品，这两个产品共同有的特征就是"生产此产品"

  ```java
  public abstract class AbstractProduct {
      public abstract void Show();
  }
  ```

  

- 3:创建抽象产品类，继承抽象产品族类，定义具体产品的公共接口

  继承上面的抽象产品族类，实现容器产品的抽象类和模块产品的抽象类

  ```java
  public abstract class MouldProduct extends AbstractProduct {
      @Override
      public abstract void Show() ;
  }
  
  public abstract class ContainerProduct extends AbstractProduct {
      @Override
      public abstract void Show() ;
  }
  ```

  

- 4：创建具体产品类，继承抽象产品类，定义生产的具体产品

  创建容器产品和模块产品，这里值得注意的是，每个厂都要创建的

  ```java
  public class ContainerProductA extends ContainerProduct{
      @Override
      public void Show() {
          System.out.println("A厂产出容器产品");
      }
  }
  public class ContainerProductB extends ContainerProduct{
      @Override
      public void Show() {
          System.out.println("B厂产出容器产品");
      }
  }
  public class MouldProductA extends MouldProduct{
      @Override
      public void Show() {
          System.out.println("A厂产出模块类产品");
      }
  }
  
  public class MouldProductB extends MouldProduct{
      @Override
      public void Show() {
          System.out.println("B厂产出模块类产品");
      }
  }
  ```

  

- 5：创建具体工厂类，继承抽象工厂类，定义常见对应具体产品实例的方法

  继承抽象工厂

  ```java
  public class FactoryA extends Factory{
  
      @Override
      public AbstractProduct ManufactureContainer() {
          return new ContainerProductA();
      }
  
      @Override
      public AbstractProduct ManufactureMould() {
          return new MouldProductA();
      }
  }
  public class FactoryB extends Factory{
  
      @Override
      public AbstractProduct ManufactureContainer() {
          return new ContainerProductB();
      }
  
      @Override
      public AbstractProduct ManufactureMould() {
          return new MouldProductB();
      }
  }
  ```

  

- 6：客户端通过实例化具体的工厂类，并调用其创建不同目标产品的方法创建不同具体产品类的实例

  ```java
  public class Main {
      public static void main(String[] args) {
          FactoryA factoryA = new FactoryA();
          FactoryB factoryB = new FactoryB();
          factoryA.ManufactureContainer().Show();
          factoryA.ManufactureMould().Show();
          factoryB.ManufactureContainer().Show();
          factoryB.ManufactureMould().Show();
      }
  }
  ```

  

## 参考文章

https://www.jianshu.com/p/7deb64f902db