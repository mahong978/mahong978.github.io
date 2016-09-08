---
layout: post
title:  "Java笔记整理：一切都是对象"
date:   2016-08-31 11:22
categories: java
tags: java
excerpt: 《Java编程思想》第二章
---

* content
{:toc}



## 引用操纵对象
 - 操纵对象的标识符是其引用
 - 引用不一定会与一个对象关联
	
## 创建对象
使用new操作符创建新对象

### 对象存储
1. 寄存器：最快的存储区，位于处理器内部，不能人为控制
2. 堆栈
 - 指针向下移动，即分配内存，向上移动，即释放内存
 - 速度仅次于寄存器
 - 编译时确定，灵活性较差
 - 对象引用位于堆栈中，对象不在堆栈中
3. 堆
 - 用于存放所有JAVA对象
 - 编译器不需要知道数据的生存期，灵活性较好
 - 内存分配和释放比堆栈要慢
4. 常量存储：常量保存在常量池，位于方法区内。垃圾回收器不会干涉方法区
5. 非RAM存储
 - 流对象：对象转换成字节流进行发送
 - 持久化对象：对象存放在ROM上
### 基本类型
不需要new进行创建
创建的是自动变量，不是引用，该变量直接存储值，置于堆栈中

|基本类型|大小|最小值|最大值|包装器类型|
|:---|:---|:---|:---|:---|
|boolean|- |-|-|Boolean|
|char|16-bit|Unicode 0|Unicode $2^{16}-1$|Character|
|byte|8 bits|-128|+127|Byte|
|short|16 bits|$-2^{15}$|$+2^{15}-1$|	Short|
|int|32 bits|$-2^{31}$|$+2^{31}-1$|Integer|
|long|64 bits|$-2^{63}$|$+2^{63}-1$|Long|
|float|32 bits|IEEE754|IEEE754|Float|
|double|64 bits|IEEE754|IEEE754|Double|
|void|-|-|-|Void|

没有无符号的数值类型
boolean类型没有确定的大小

### 数组
JAVA确定数组一定会被初始化，而且不能越界访问
对象初始化为null，基本数据初始化为0

## 永远不需要销毁对象
### 作用域
作用域由花括号框定
作用域内不能有子作用域
离开作用域后，对象依然存在，会由垃圾回收器销毁

## 类
基本类型成员会自动被初始化
基本成员默认值
|基本类型|默认值|
|:---------|:------|
|boolean|false|
|char|'\u0000'(null)|
|byte|(byte)0|
|short|(short)0|
|int|0|
|long|0L|
|float|0.0f|
|double|0.0d|

局部变量不会被初始化（未初始化变量会引发编译错误）
	
## 方法
传递的参数实际上是引用


----------
JAVA没有“向前引用”问题，即类和方法的使用与定义位置无关
java.lang是默认导入到java文件中的