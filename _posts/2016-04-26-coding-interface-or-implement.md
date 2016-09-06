---
layout: post
title:  "面向接口编程与面向实现编程"
date:   2016-04-26 17:02
categories: 编程
tags: 编程 设计模式
---

* content
{:toc}

最近拜读了四人组的经典名作*《设计模式 可复用面向对象软件的基础》*一书，打算以博客的形式进行笔记与思考

书中提到了可复用面向对象设计的原则，其中的第一个原则是：

> 针对接口编程，而不是针对实现编程

其实在使用面向对象语言进行编程的时候，经常不经意间就会涉及书中提到的知识，但是一旦用专门的词汇进行描述时，第一反应就是“诶？这是什么意思？看不懂啊”，只有经过反复的阅读与思考，才意识到这是日常编程经常遇到的问题，并且得到更深的理解




--------------
## 面向实现编程

举个例子，假设有两种品牌的轮胎，普利司通（Bridgestone）和米其林（Michelin），而轮胎的共同特性都是会转（roll）。那么我们可以得到两个类：

``` java
class Bridgestone {
    public void roll() {
        System.out.print("Bridgestone is rolling.");
    }
}

class Michelin {
    public void roll() {
        System.out.print("Michelin is rolling");
    }
}
```

对于一辆装了普利司通轮胎的汽车（Car），汽车的转动（roll）就是轮胎的转动：

``` java
class Car {
    public void roll(Bridgestone tire) {
        tire.roll();
    }
}
```

那如果我装了米其林的轮胎呢？

``` java
Car car = new Car();
Michelin tire = new Mechilin();
car.roll(tire);
```

显而易见，程序将会出错。这就是面向实现编程，变量是指向特定类的实例的。
这种强烈的依赖关系将会大大地抑制编程的灵活性和可复用性。

----------------------------------

## 面向接口编程

如果将这两种轮胎的共同特性提取出来，在转动轮胎的时候，只关注“是轮胎”本身，而不去了解“是什么品牌的轮胎”，问题就迎刃而解了

``` java
interface Tire {
    public void roll();
}

class Bridgestone implements Tire {
    public void roll() {
        System.out.print("Bridgestone is rolling.");
    }
}

class Michelin implements Tire {
    public void roll() {
        System.out.print("Michelin is rolling");
    }
}
```

接口Tire定义了“转动”这个接口，但把实现延迟到了子类中

``` java
class Car {
    public void roll(Tire tire) {
        tire.roll();
    }
}

Car car = new Car();
BridgeStone tire1 = new Bridgestone();
Michelin tire2 = new Mechilin();
car.roll(tire1);
car.roll(tire2);
```

汽车在转动时，并不关注“是什么品牌的轮胎“，只关注”是轮胎“，想怎么转就怎么转

从这个例子我们可以看出，面向接口编程的特性：

> 客户无须知道他们使用对象的特定类型，只须对象有客户所期望的接口

客户使用汽车转动轮胎时，无须知道轮胎的特定类型（品牌，对应的子类），只要轮胎有客户所期望的接口（roll）就行了

> 客户无须知道他们使用的对象是用什么类来实现的，他们只须知道定义接口的抽象类

客户使用Car的roll方法调用轮胎对应的方法时，不需要知道这个轮胎实例是用什么子类实现的，他只需要知道定义转动方法的抽象类（JAVA中的interface）的内容就行了

----------------------------------------

## 总结
面向接口的编程方式是面向对象设计的一个原则，使用这种编程思想，我们可以容易地写出具有可复用性的代码，这对于代码的理解和维护具有很大的帮助