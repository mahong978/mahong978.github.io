---
layout: post
title:  "Java笔记整理：访问权限控制"
date:   2016-09-04 11:19
categories: java
tags: java
excerpt: 《Java编程思想》第六章
---

* content
{:toc}



访问权限控制的等级，从大到小依次为：
public，protected，包访问权限，private


----------

## 包
包是库单元
### 代码组织
一个JAVA源代码文件是一个编译单元（转译单元）
编译单元内有一个或零个public类，该类名必须和编译单元名字相同
编译单元内其他的类不能为public

JAVA可执行程序是class文件
如果使用package语句，必须是文件非注释的第一句代码
### 包名
包名的第一部分是类创建者的反序域名
整个包名为小写
包路径中除包名给出的相对路径外的部分，在环境变量CLASSPATH给出
使用JAR则必须在CLASSPATH中给出完整路径
### import static
使用import static可以导入对应类中的静态方法，可用于定制工具库
### 用import改变行为
相当于条件编译，不同包下有相似的类，那么可以通过修改import的包来改变程序的行为



## JAVA访问权限修饰词
无论如何，所有事物都具有某种形式的访问权限控制

## 包访问权限
对于包外的类相当于 private
包访问权限为包的存在具有意义

取得某成员的访问权限的途径
	1. 使其成为public
	2. 不加访问权限修饰词，并与其类置于同一包内
	3. 继承类，可以获得public和protected成员的访问权限，private不行，包访问权限的只能包内继承才行
	4. 提供get/set方法
## public
没有指定package的类，全部归属于对应目录下的默认包中
## private
一个使用方法是让构造器私有化，控制对象的创建
## protected
处理的是继承的概念
## protected也提供包访问权限

# 接口和实现
访问权限控制相当于具体实现的隐藏
数据和方法放进类中 + 具体实现的隐藏 = 封装

# 类访问权限
public：希望某个类可以被客户使用
注意：
1. 每个编译单元只能有一个public类
2. public类的名称必须和编译单元文件名相同，大小写区分
3. 编译单元可以没有public类，此时命名随意

不是public的类具有包访问权限
类没有private和protected情况
如果某个包访问权限的类含有public的静态成员，客户可以使用这个静态成员，但不可以生成该类的对象
