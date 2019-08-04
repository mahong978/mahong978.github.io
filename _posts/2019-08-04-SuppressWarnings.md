---
layout: post
title:  "SuppressWarnings"
date:   2019-08-04 21:24
categories: Java
tags: Java
---

* content
{:toc}


表示让编译器不对所注解的元素提示警告

注解的元素可以是类型，域，方法，参数，构造器，本地变量

比如

```
@SuppressWarnings("unchecked")
public static void main(String[] args) {
    List list = new ArrayList<>();
    list.add("1213");
}
```

可以填写多种类型，用数组的形式

注解的元素应该越深越好

重复的和未能被识别的警告名会被忽略，支持的警告名视编译器和IDE而异

以IntelliJ为例，`Alt + Enter`会有对应的抑制警告的选项