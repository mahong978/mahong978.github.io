---
layout: post
title:  "Firebase初探：配置"
date:   2016-08-24 02:52
categories: Android
tags: Android Firebase
---


* content
{:toc}


当app发展到本地以外，就不得不牵扯到服务器了，当然这对于大公司来说并不是什么大问题，服务器有后台人员维护，网站有前端人员负责，移动端也有对应人员搞定。但是服务器就一个（或一组，一间etc），其中的数据要联动到web，android，ios等不同平台上，可能就会有不同的差异，开发工具包自然是一点，从服务器拉取的方式和传送数据的格式可能也会有不同。

对于小型开发团队或者独立开发者来说就更痛苦了，在开发应用的同时还需要抽身去创建和维护服务器，即使现在已经有很多的云服务供应商，但是这点还是避免不了。
Firebase作为提供实时后端数据库的公司，它可以帮助快速地开发网页端和移动端的应用，自从Google收购了之后，在今年的IO大会上大放异彩，它在使原来的服务更加方便之外，还加入了Google的各种新元素，包括远程配置，云上调试，崩溃报告，动态链接等等，基本上所有的功能都只要几行代码就能完成，而惊喜的是，这些工具都是跨平台的，这意味着代码的移植变得很简单。

另外Firebase Cloud Messaging也是Google Cloud Messaging的扩展版，随着Google对中国的兴趣越来越大，将来说不定还是会回归的，再加上Android N中加强版的Doze mode强调了GCM的重要性，因此Firebase在未来Android应用的作用将不可小觑。

下面的几篇博文将关于Firebase的使用进行学习。






-------------
配置当然是第一步。先来讲讲服务器的配置

## 服务器
先来到[Firebase的官网](https://firebase.google.com)上，右上角Go to console转入控制台：
![](http://img.blog.csdn.net/20160824023439848)

然后新建项目：
![](http://img.blog.csdn.net/20160824023653101)
设置国家地区并不是选择服务器的位置，而是为后面的统计功能提供币种的参考，当然还代表了公司或组织所在的位置

创建成功后就会进入该项目的控制台，接下来就是为这个项目添加一个应用了，这里以Android应用为例：
![](http://img.blog.csdn.net/20160824024044451)
我们需要提供package的名称，这可以在app级别的gradle配置文件，或者是Manifest.xml中查看，。然后是调试key store的SHA1码，这个可以通过开启终端，输入下面的命令行查看：

```
keytool -exportcert -list -v -alias <key名> -keystore <keystore路径>
```

如果只是默认的调试，key名是androiddebugkey，keystore名是debug.keystore，路径是在用户文档的.android文件夹下（Windows平台）

添加应用成功后浏览器将会自动下载google-services.json文件，这将用来配置对应的Android工程


----------
## Android工程
### 准备条件
设备要求：Android2.3或以上，Google Play服务9.4.0或以上
开发要求：Google Play Serivce SDK，Android Studio 1.5或以上

首先在项目级的build.gradle中添加对Google Service的依赖：

```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.1.3'
        classpath 'com.google.gms:google-services:3.0.0'
    }
}
```

然后在应用级的build.gradle中添加Google Service的插件：

```
apply plugin: 'com.android.application'

android {
    //...
}

dependencies {
    //...
}

apply plugin: 'com.google.gms.google-services'
```

最后就是把之前得到的google-services.json放到对应的app目录下：
![](http://img.blog.csdn.net/20160824131617756)

Sync一下，通过就说明配置成功了


----------
以上就是Firebase的初步配置。以后将会对Firebase提供的各种服务进行介绍