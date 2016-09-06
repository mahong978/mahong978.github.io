---
layout: post
title:  "Firebase初探：实时数据库(1)"
date:   2016-08-28 16:57
categories: Android
tags: Android Firebase
---

* content
{:toc}

既然Firebase是做后端数据库起家的，那就自然少不了实时数据库这一块了

实时数据库，顾名思义，就是不需要客户端自己去轮询，而是每当监听的区域发生变动时，数据库都会通知客户端去更新，设备将会在毫秒级的时间内接收到数据更新

除此之外，考虑到写数据时遇到的无网络连接问题，Firebase的数据库API使用了本地缓存，使得在离线状态下也能保持读写不失败，并且会在网络恢复连接时和服务器进行同步

和常见的数据库服务器不同，Firebase使用JSON存储数据，这使得数据结构非常可观






----------
## 数据结构

如前面所说，Firebase使用了JSON树来存储数据，这使得数据结构更加可观，而且也不需要考虑数据类型

但是这也带来了一个问题。我们平时构造JSON数据，多层嵌套的写法是很常见的事，以社区留言记录举例：

```
"record" {
  "record1": {
    "title": "TITLE1"
    "content": "..."
    "writer": "S"
    "reply": {
      "reply1": {
        "sender": "S1",
        "content": "..."
      }
      "reply2": {
      ...
    }
  },
  "record2": {
  ...
}
```

对于该在线社区的首页来说，需要列举出各个留言的标题，但是对于多层嵌套的写法来说，就需要将整个record树下载下来了，这对于效率来说是很不利的

Firebase推荐使用平展数据结构，说白了就是将各个属性分开成不同的树来存放，一个表拆成多个表，也就是反规范化。上面的例子就可以这样写：

```
"record": {
  "record1": {
    "title": "TITLE1"
    "content": "..."
    "writer": "S"
  }
  "record2": {
  ...
}

"reply": {
  "record1": {
    "reply1": {
      "sender": "S1",
      "content": "..."
    }
    "reply2": {
    ...
  }
  "record2": {
  ...
}
```

这样就能保证读取时的效率了

还是存在反规范化不能解决的问题，例如双向关系，对于一个用户来说可以从属于一个群组，一个群组又包含了用户，这时可以使用索引来解决：

```
"user": {
  "u1": {
    groups: {
      "g1": true
      ...
    }
  }
  ...
}

"group": {
  "g1": {
    members: {
      "u1": true
      ...
    }
  }
}
```

也就是说，存在双向关系的两个元组，将对方的元素作为”索引“来使用。这样就使得在访问这样的数据时可以保证效率，当然这里就要牺牲了一点数据冗余度了

## 基本使用

进入控制台后，选择database，可以看到右侧窗口有“数据”，“规则”和“使用情况”三个选项卡，“数据”会对JSON树进行图形化显示，“使用情况”则是对数据库的带宽，存储空间，连接数和总发送字节进行统计。关键是“规则”选项，在这里，可以使用类似JavaScript的语法，对数据库的读写权限，数据规则和索引进行设置

Firebase数据库的规则主要分四种：
|规则|说明|
|:-----|:------|
|.read|允许用户读取数据的条件|
|.write|允许用户写入数据的条件|
|.validate|定义值的数据类型，格式等条件|
|.indexOn|定义加入索引的项|

### 读写授权
数据库的默认rule是：

```
{
  "rules": {
    ".read": "auth != null",
    ".write": "auth != null"
  }
}
```

这里表示根节点读写的条件是认证不为空，说明需要身份验证才能获得权限（auth是规则的预定义变量，后面会讲到）

对应字符串中的逻辑计算结果决定了此节点下的读写权限

.read和.write是级联的，也就是说，父节点的读写权限直接决定子节点的读写权限，子节点的设置将被忽略

```
{
  "rules": {
    "father": {
      ".read": false,
      "child": {
        ".read": true
      }
    }
  }
}
```

由于父节点禁止了读取，即使子节点允许读取，也是没有用的

### 数据验证
.validate将会对新写入的数据进行验证，验证通过的数据方可写入。因此.validate发挥作用需要.write为true，另外，因为.validate是检查新数据，因此常需要和newData变量搭配使用（newData是预定义常量，表示写入操作后应有的数据，包含新数据和未改动数据）

```
{
  "rules": {
    "age": {
      ".validate": "newData.isNumber() && newData.val() >= 0"
    }
  }
}
```

有关上面的函数问题，下面会讲到，可以看出年龄要求写入的数据是数字，并且值的大小不小于0

.validate是不级联的，只要新数据的任意一个子节点不满足验证要求的话，数据就不能写入

#### 预定义变量
为了方便引用数据，用户认证等信息，数据库规则提供了如下预定义变量
|预定义变量|说明|
|:-----------|:----|
|now|从时间纪元起到当前时间的毫秒数|
|root|RuleDataSnapshot，当前操作所在路径的根目录|
|newData|RuleDataSnapshot，表示写入操作后应有的数据，包含新数据和未改动数据|
|data|RuleDataSnapshot，表示原先的数据|
|$variable|获得对应的节点名|
|auth|表示通过验证的用户身份|

其中root，newData和data是RuleDataSnapshot类型，从名字可以看出，这些都是规则所用的数据快照，因此我们可以使用相关的一些函数获得其中的信息：
|RuleDataSnapshot函数|说明|
|:------------------------|:-----|
|val()|读取明确的子节点内容，这要求需要先用child()定位到叶子节点，然后再用val()读取内容。直接对一块数据进行val()不会返回想要的“一块数据”|
|child()|返回参数所给的相对路径，所对应的节点的快照数据|
|parent()|返回当前节点的父节点，所对应的快照数据。如果父节点不存在，会导致这条规则直接失败|
|hasChild(path)|如果这个子节点存在返回true|
|hasChildren([children])|如果子节点列表都存在返回true|
|exists()|如果此数据快照中有数据则返回true|
|getPriority()|返回此数据快照中数据的优先级|
|isNumber()|如果其中的数据是数字，返回true|
|isString()|如果其中的数据是字符串，返回true|
|isBoolean()|如果其中的数据是布尔类型，返回true|

另外还支持字符串函数和各种操作符，详情可参阅[https://firebase.google.com/docs/reference/security/database/](https://firebase.google.com/docs/reference/security/database/)

#### $ 变量
使用$变量可以获得当前节点的名字的字符串，例如：

```
{
  "rules": {
    "students": {
      "$stu_id": {
        "name": {
          ".write": "$stu_id.contains('sysu')"
        }
      }
    }
  }
}
```

上面的规则说明，对于修改一个学生的姓名，要求该学生的id中必须包含“sysu”。

需要注意的是：
1. 这条规则将会对students节点中的所有子节点有效
2. $变量所得到的都是字符串

#### auth
如果用户是经过身份认证的，那么auth变量将不会是null，auth的属性有：
|属性|说明|
|:----|:----|
|provider|所使用的身份认证方法，passsword，anonymous，google，facebook，twitter或github|
|uid|用户ID|

使用auth可以确保只有用户本人才能访问其信息：

```
{
  "rules": {
    "users": {
      "$user_id": {
        ".read": "$user_id === auth.uid",
        ".write": "$user_id === auth.uid"
      }
    }
  }
}
```

### 索引

使用.indexOn来规划索引，索引可以帮助提高检索性能：

```
{
  "rules": {
    "students": {
      "indexOn": ["name", "age"]
    }
  }
}
```

服务器设置了.indexOn规则后，客户端使用orderByValue()向服务器检索学生信息，就能获得更好的性能


----------
以上就是Firebase实时数据库在服务器方面的信息，可以看到，Firebase的后端服务器完全不需要开发者搭建，开发者只需要管理好数据结构以及相关规则即可。

在数据结构方面，对于Firebase所使用的JSON树结构，需要注意一下不要使用多层嵌套的写法，而是进行反规范化，虽然损失了冗余度，但对于性能提升来说是必要的。Firebase还支持JSON导入和导出，这也使得数据的转移非常方便。

在规则方面，Firebase使用了JavaScript的写法，设置起来也是相当方便的。