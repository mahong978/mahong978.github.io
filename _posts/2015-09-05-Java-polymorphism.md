---
layout: post
title:  "Java笔记整理：多态"
date:   2016-09-05 11:50
categories: java
tags: java
excerpt: 《Java编程思想》第八章
---


* content
{:toc}



多态是数据抽象和继承之后的第三种基本特征

利用多态可以创建可扩展的程序

多态可以消除类型之间的耦合关系


## 向上转型
向上转型会将接口缩小至基类的大小


## 方法调用绑定

绑定：将一个方法调同一个方法主体关联起来

 - 前期绑定：程序执行前进行绑定（编译器和连接程序），默认的绑定方式
 - 后期绑定：程序运行时根据对象的类型进行绑定，又叫动态绑定和运行时绑定
	
JAVA中除了static方法和final方法之外，所有方法都是后期绑定

帮助设计可扩展的程序
	
多态是一项将改变的事物和未变的事物分离开来的重要技术


## 缺陷
	
### 覆盖私有方法
覆盖private方法将会产生一个全新的方法
		
### 域与静态方法
只有普通的方法调用可以是多态的

域的访问是编译时解析的，导出类默认访问自己的域，要访问基类的域要使用关键字super

静态方法是与类而不是对象相关联的


## 构造器
构造器实际上是static方法，因此不具有多态性

基类的构造器总是在导出类的构造过程中被调用，并且沿着继承层次向上链接

如果没有指定调用哪个构造器，默认调用基类的默认构造器

构造器调用顺序：

1. 调用基类构造器
2. 按声明顺序初始化成员
3. 调用导出类构造器主体
	
### 销毁
销毁的顺序应该和初始化顺序相反，即声明的初始化顺序

先对导出类进行清理，然后才是基类

如果有成员对象是被共享的，必须使用引用计数
		
遍写构造器时避免使用其他方法，唯一能安全调用的是基类中的final方法


## 继承

继承设计准则：继承表达行为间差异，用字段表达状态上的变化

### 纯继承与扩展

导出类具有基类的接口

纯继承：导出类可以完全代替基类

导出类中扩展接口部分不能被基类访问

### 向下转型

所有转型都会得到检查，运行时进行检查，如果不是会返回ClassCastException（运行时类型识别RTTI）