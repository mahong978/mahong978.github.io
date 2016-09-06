---
layout: post
title:  "Firebase初探：实时数据库(2)"
date:   2016-08-29 14:37
categories: Android
tags: Android Firebase
---


* content
{:toc}

前面的那篇介绍了Firebase实时数据库的相关知识，那么客户端将如何与实时数据库进行沟通呢？





------------
## 小试牛刀
Firebase的SDK当然是必不可少的了，为了使用数据库相关的API，需要在应用的依赖项中添加：

```
compile 'com.google.firebase:firebase-database:9.4.0'
```

为了和数据库交互，需要得到数据库的一个实例，而且具体的交互对象是数据库中的某个节点，这时就需要获得这个节点的一个引用：

```java
FirebaseDatabase database = FirebaseDatabase.getInstance();
DatabaseReference mRef = database.getReference("students");
```

通过这两句代码，我们得到了数据库的实例和节点的引用，数据的读写操作都是用DatabaseReference进行的

那么数据库基本的增删查改如何进行呢？我们先来看看最直观的写和读

## 写
写操作包括了增和删，但是SDK并没有对这两个操作进行明显的区分，而是将其视为写操作，SDK中有四种写操作：
|方法|说明|
|:----|:----|
|setValue()|在指定的路径，将数据写入或替换|
|push()|生成唯一ID，路径进入到此ID中|
|updateChildren()|更新此路径中的部分键值，而不是所有数据|
|runTransaction()|进行并发更新|

### setValue
setValue(Object)是最基本的写操作。得到某个节点的引用后，调用setValue()即可写入数据，如果该节点原本就有数据，则会覆盖该节点下的所有数据
由于数据库使用了JSON作为数据格式，setValue()可接受的参数类型有：

 - Boolean
 - Long
 - Double
 - Map< String, Object >
 - List< Object >

除了基本的数字和布尔值外，还可以接受自定义的对象，自定义对象的要求有：

 - 类必须要有默认构造器
 - 类必须为成员变量提供getter方法。没有getter方法的成员变量对应的键值将会被设为缺省值

可以看到，setValue()支持Object列表，这对于同时插入多个数据很有帮助，另外还支持Map< String, Object >，利用映射的特点，我们可以将节点路径作为键，要插入的数据作为值，就可以做到在多个位置同时进行写入了

如果希望在指定节点下进行写入，可以使用child()方法先定位到该节点下再写入，child()方法返回DatabaseReference对象，因此可以迭代使用

```java
mRef.child("students").setValue(student);
```

setValue()方法将会返回一个Task对象，之前的身份认证一章介绍过Task了，因此可以给其添加一个OnCompleteListener监听器，检查数据是否成功写入

### push
push()方法将会在当前节点下添加一个新的子节点，该子节点的键基于时间戳，因此是唯一的。push()创建了子节点后，会返回该子节点的引用

### updateChildren
updateChildren()方法可以修改部分数据，而不影响该节点下的其他数据，只要在参数对象中提供需要修改的数据就行了
Firebase推荐使用平展开的数据结构，一个对象的属性可能会分散在多个JSON树中，因此对于这种数据扇出的情况，使用updateChildren(Map< String, Children >)可以很方便地一次性更新分散开来的数据，这种方式的更新具有原子性

### runTransaction
事务是数据库的重要课题之一。对于一篇帖子，一个用户点一次赞，数据库中对应的点赞数就要加一，这中间的过程为：客户端查找并获得该帖子的点赞数，加一，然后更新该帖子的点赞数。那么问题就来了，如果多个用户进行点赞时就需要考虑数据的同步问题
先来看看runTransaction的用法：

```java
mRef.runTransaction(new Transaction.Handler() {
    @Override
    public Transaction.Result doTransaction(MutableData mutableData) {
        Post post =  mutableData.getValue(Post.class);
        post.like = post.like + 1;

        mutableData.setValue(post);
        return Transaction.success(mutableData);
    }

    @Override
    public void onComplete(DatabaseError databaseError, boolean b, DataSnapshot dataSnapshot) {
        // ...
    }
});
```

调用runTransaction()方法需要提供一个Handler对象，并实现其中的doTransaction()方法和onComplete()方法。
更新数据的逻辑就是写在onTransaction()方法中，方法有一个MutableData对象参数，顾名思义，它是引用的数据，而且是可变的，这意味着MutableData的变动对于数据库来说时可视的，反之亦然。
因此这就解决了事务问题，我们通过MutableData取得最新的数据，修改它，然后写回到MutableData中，万事大吉。
如果这次事务失败了怎么办？服务器会告知客户端，客户端会重新进行事务，直到成功或者重试次数过多为止

## 读
由于Firebase是实时数据库，检索数据将不是传统的轮询形式，而是数据库主动告知你数据的变化，因此SDK提供的检索数据不是read形式，而是使用监听器来实时获得更新数据
Firebase提供的监听器有两种：ValueEventListener，ChildEventListener

### ValueEventListener
该监听器将在设置时触发一次，然后会在未来值发生变化时触发
实现该监听器需要实现其onDataChange()方法和onCancelled()方法。
当引用对应的路径下发生数据改变时，便会触发onDataChange()回调方法，该方法将提供给定路径下的数据快照，因此尽量监听低层节点，不要在高层节点设置监听器，以防拉取大型的数据快照
当数据访问失败或者由于安全规则被拒绝时，事件将被取消，这时会回调onCancelled()方法

```java
ValueEventListener listener = new ValueEventListener() {
    @Override
    public void onDataChange(DataSnapshot dataSnapshot) {
        Post post = dataSnapshot.getValue(Post.class);
        // ...
    }

    @Override
    public void onCancelled(DatabaseError databaseError) {
        // ...
    }
};
mRef.addValueEventListener(listener);
```

代码中使用了addValueEventListener()方法添加监听器，还有另一个方法addListenerForSingleValueEvent()，这种方法设置的监听器将只会触发一次，因为ValueEventListener会在被设置到引用上时触发一次，因此addListenerForSingleValueEvent()可用于UI的初始化或者不需要持续监听的情况

### ChildEventListener
ChildEventListener将会对该节点下的各种子节点变化进行监听，包括添加，移除，更新和移动

 - onChildAdded()：该方法会对当前的所有子节点都触发一次，并且会在未来添加子节点时触发。因此这个方法可用于遍历当前节点
 - onChildChanged()：每当有子节点被修改，该方法被触发
 - onChildRemoved()：每当有子节点被删除，该方法被触发
 - onChildMoved()：每当子节点的次序发生变化时（例如修改优先级），该方法被触发

以上方法除了onChildRemoved()没有String参数外，都有一个DataSnapshot参数和一个String参数，前者表示被添加/修改后/被移除/被移动的节点的数据，后者表示该节点前一个节点的名字

### 移除监听器
引用调用removeEventListener(ValueEventListener)或removeEventListener(ChildEventListener)，即可移除指定的监听器
如果在同一路径设置多个监听器，则需要一个一个移除掉，因此尽量不要在addValueEventListener()和addChildEventListener()直接使用无引用的匿名内部类
父节点移除监听器不会影响到子节点的监听器

### 排序和过滤
Firebase也提供了排序和过滤的基本功能。DatabaseReference是继承自Query类的，Query提供了各种排序和过滤的方法
#### 排序
提供的排序方法有：

|方法|说明|
|:---|:---|
|orderByChild()|按指定子节点的值进行排序|
|orderByKey()|按键进行排序|
|orderByValue()|按值进行排序|
|orderByPriority()|按优先级进行排序|

各方法的排序策略为：

 - orderByChild
     按值为null，false，true，数值（升序），字符串（升序），对象的顺序排列，值相同的和值为对象的以键名按字典顺序排列
 - orderByKey
     如果键可以解析为32位整数的，以升序排在前面
     不能解析的字符串键以字典顺序排在后面
 - orderByValue
     排序策略与orderByChild相同
 - orderByPriority
     次序为：
      - 没有设置优先级的在最前面
      - 以数字为优先级的
      - 以字符串为优先级的

    有冲突的一律再按键排序
    
一个引用只能使用一种排序方法
引用调用了排序方法后，返回的是Query类型引用


#### 过滤
提供的过滤方法有：

|方法|说明|
|:---|:---|
|limitToFirst()|从排序结果开始的最大项目数|
|limitToLast()|从排序结尾开始的最大项目数|
|startAt()|返回等于或大于指定键，值或优先级的项目|
|endAt()|返回小于或等于指定键，值或优先级的项目|
|equalTo()|返回等于指定键，值或优先级的项目|

其中startAt()，endAt()， equal()可以指定键，值或优先级，因此有很多重载版本

## 删
读和写介绍了增，改和查的方法，还少了删，其实删很简单，就是数据库引用调用removeValue()方法即可，也可以通过设置值为null的方式进行删除

## 离线
### 数据持久化
Firebase也在数据持久性上下了功夫
如果有数据需要写入数据库，而此时失去了网络连接，应用仍可继续运行，写操作队列会保存在缓存中，连接恢复后继续写入数据库。那如果接着用户关闭了应用呢？不用着急手动保存，开启磁盘持久化功能可以让数据保存在ROM中，即使应用重启也不会丢失。
开启磁盘持久化功能只需要一句代码：

```java
FirebaseDatabase.getInstance().setPersistenceEnabled(true);
```
另外，持久化功能也可以保存用户的身份验证

### 保持数据更新
Firebase提供了一种不用添加监听器的方式来保持数据更新：

```java
DatabaseReference stuRef = FirebaseDatabase.getInstance().getReference("students");
stuRef.keepSynced(true);
```

这样的话，系统将会自动地与数据库保持数据同步
默认情况下，客户端可以在本地缓存10MB的数据，一旦数据量超过10MB，将会根据LRU删除缓存数据来腾出空间。但对于开启keepSynced的数据，不会被从缓存中删除

### 断开连接
有时我们需要在客户端与数据库断开连接时，在数据库上标记该用户已断开的情况，对于这种情况，Firebase提供了相关方法：

```java
DatabaseRef presenceRef = FirebaseDatabase.getInstance().getReference("userLog");
presenceRef.onDisconnect().setValue("Disconnected!");
```

通过调用onDisconnect()，客户端会通知服务器，服务器检查用户的操作是否正确，然后通知客户端是否合法，如果onDisconnect成功创建，服务器会持续监听客户端的连接状态，一旦客户端断开连接，服务器就会执行设定的操作
可以设置的操作不止set，update和remove都可以
可以通过添加CompletionListener来知道onDisconnect是否创建成功
调用cancel()就可以取消onDisconnect事件

### 监听连接状态
客户端在数据库实例中加了一个“/.info/connected”的节点，该节点会根据连接状态而持续更新，因此可以监听这个节点的值变化，来达到监听连接状态的效果

### 延迟
客户端时间和服务器时间不一定相同，两者之间存在时间偏差，因此我们在记录一些跟时间有关的数据，尽量使用ServerValue.TIMESTAMP，而不是本地时间


----------
以上就是关于客户端有关的Firebase数据库SDK使用介绍。通过两章关于Firebase实时数据库的介绍，我们可以大致地了解了实时数据库的使用和相关原理，总结也没什么好说的了，总之Firebase的数据库是个很强大的东西，不需要手动配置，不需要主动轮询，甚至也不要对数据库进行多少维护，这对于独立开发者来说确实是一大神器