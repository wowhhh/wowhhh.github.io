---
layout: post
title:  "工厂方法模式 :)"
tags:   创建型模式
date:   2019-06-05 00:00:00 +0800
categories: [设计模式]


---

## 简单介绍

工厂方法模式是简单工厂的仅一步深化， 在工厂方法模式中，我们不再提供一个统一的工厂类来创建所有的对象，而是针对不同的对象提供不同的工厂。也就是说每个对象都有一个与之对应的工厂。

## 应用场景

同简单工厂模式类似，因为工厂方法模式就是简单工厂模式的改进。

## 优缺点

- 优点

  >- 在工厂方法模式中，工厂方法用来创建客户所需要的产品，同时还向客户隐藏了哪种具体产品类将被实例化这一细节，用户只需要关心所需产品对应的工厂，无须关心创建细节，甚至无须知道具体产品类的类名。
  >- 基于工厂角色和产品角色的多态性设计是工厂方法模式的关键。它能够使工厂可以自主确定创建何种产品对象，而如何创建这个对象的细节则完全封装在具体工厂内部。工厂方法模式之所以又被称为多态工厂模式，是因为所有的具体工厂类都具有同一抽象父类。
  >- 使用工厂方法模式的另一个优点是在系统中加入新产品时，无须修改抽象工厂和抽象产品提供的接口，无须修改客户端，也无须修改其他的具体工厂和具体产品，而只要添加一个具体工厂和具体产品就可以了。这样，系统的可扩展性也就变得非常好，完全符合“开闭原则”。

- 缺点

  >- 在添加新产品时，需要编写新的具体产品类，而且还要提供与之对应的具体工厂类，系统中类的个数将成对增加，在一定程度上增加了系统的复杂度，有更多的类需要编译和运行，会给系统带来一些额外的开销。

## 构建要素

|          组成(角色)          |                关系                |                         作用                          |
| :--------------------------: | :--------------------------------: | :---------------------------------------------------: |
|      抽象产品(Product)       |           具体产品的父类           |                描述具体产品的公共接口                 |
| 具体产品（Concrete Product)  | 抽象产品的子类；工厂类创建的目标类 |                  描述生产的具体产品                   |
|     抽象工厂（Creator）      |           具体工厂的父类           |                描述具体工厂的公共接口                 |
| 具体工厂（Concrete Creator） |     抽象工厂的子类；被外界调用     | 描述具体工厂；实现FactoryMethod工厂方法创建产品的实例 |

## 如何实现

- 创建抽象工厂类，定义具体工厂的公共接口
- 创建抽象产品类 & 定义具体产品的**公共接口**
- 创建具体产品类（继承抽象产品类） & 定义生成的具体产品
- 创建具体工厂类（继承抽象工厂类），定义创建对应具体产品实例的方法
- 外界调用具体工厂类的方法，从而创建不同具体产品类的实例

### 例子

> 还是接上文的实例
>
> 现在烩面面馆老板要做西安面种了，就比如裤带面和旦旦面吧，如果在当前的烩面面馆售卖制作这种面会比较困难，包括制作和销售方面，所以现在决定置办个西安面馆做这件事情。
>
> 冲突：当前的烩面面馆售卖制作这种面会比较困难，包括制作和销售方面
>
> 解决方案：置办个西安面馆做这件事情。

代码示例：

- 1：创建抽象工厂类，定义具体工厂的公共接口

  ```java
  public abstract class Factory {
      public abstract Noodles sell();
  }
  ```

- 2：创建抽象产品类 & 定义具体产品的**公共接口**

  ```java
  public abstract class Noodles {
      public abstract void done();
  }
  ```

- 3：创建具体产品类（继承抽象产品类） & 定义生成的具体产品

  ```java
  public class KuDaiNoodles extends Noodles{
      @Override
      public void done() {
          System.out.println("Done of KuDai Noodles");
      }
  }
  
  public class DanDanNoodles extends Noodles{
      @Override
      public void done() {
          System.out.println("Done of DanDan Noodles");
      }
  }
  ```

- 4：创建具体工厂类（继承抽象工厂类），定义创建对应具体产品实例的方法

  ```java
  public class DanDanFactory extends Factory {
      @Override
      public Noodles sell() {
          return new DanDanNoodles();
      }
  }
  
  public class KuDaiFactory extends Factory{
      @Override
      public Noodles sell() {
          return new KuDaiNoodles();
      }
  }
  ```

- 5：外界调用具体工厂类的方法，从而创建不同具体产品类的实例

  ```java
  public class FactoryMethodPatternTest {
      public static void main(String[] args) {
          DanDanFactory danDanFactory = new DanDanFactory();
          danDanFactory.sell().done();
  
          KuDaiFactory kuDaiFactory = new KuDaiFactory();
          kuDaiFactory.sell().done();
      }
  }
  ```




## 源码

![image-20210323101908035](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2021-03-16/image-20210323101908035.png)

## 参考文章

https://blog.csdn.net/carson_ho/article/details/52343584

https://design-patterns.readthedocs.io/zh_CN/latest/creational_patterns/factory_method.html#id25

https://www.hollischuang.com/archives/1408