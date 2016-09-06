---
layout: post
title:  "Java笔记整理：控制执行流程"
date:   2016-09-02 10:48
categories: java
tags: java
excerpt: 《Java编程思想》第四章
---

* content
{:toc}




## true和false
所有条件语句都用true和false决定执行路径
JAVA不允许将一个数字作为布尔值使用

## if-else
else if不是关键字

## 迭代
JAVA中唯一使用到逗号操作符的地方，是for循环的控制表达式
无论是初始化还是步进部分，语句都是顺序执行的，而不是同时进行

## foreach
体现了封装的特性
更加简洁的for语法

## goto
JAVA使用标签进行跳转
break和continue可配合标签使用，效果是先跳到标签对应的循环部分，然后再进行break/continue操作
JAVA使用标签的唯一理由是希望在多层嵌套中break或continue

## switch
只能对int，String或enum类型进行操作

