---
layout: post
title:  "Android N新特性：direct reply notification"
date:   2016-09-08 16:40
categories: Android
tags: Android Notification
---

* content
{:toc}


Android N带来了一系列新功能，比如分屏与画中画，更好的后台管理等，其中就包括了direct reply notification，顾名思义，就是可以直接回复的通知。









试想一下，在以往的Android系统中，你正在使用的一款社交应用不在前台时，当有新消息时往往是以通知的方法进行提醒的，这时你从状态栏下滑，显示出通知列表，然后点击那个通知进入应用的聊天界面，最后才进行回复。这中间包括了多个手势和界面跳转，当应用优化或者手机性能较差时，这将会花上相当的时间，所带来的用户体验将会是比较差的。

Android N带来的直接回复通知的功能，允许用户直接在通知控件上进行回复，这就免去了上面所述的步骤，优化了用户体验。下面就让我们对direct reply notification进行研究


----------
## 主要流程

在开始之前，先来理清一下中间的步骤：

1. 接收消息一般是使用Service在后台运行
2. 当有新消息时，Service发出Notification
3. 用户在通知上回复后，需要进行发送，由于通知的响应是通过PendingIntent进行的，所以这里使用IntentService来负责发送是最好的
4. IntentService发送完后，发送广播，Service监听相关广播，收到广播后进行通知的更新

主要流程大概就是这样，下面就开始一步步地来进行探索


----------
## 构建Notification

监听消息的Service在接收一条新消息后，将发出一条通知，那么直接回复的通知是如何构建的呢？

这里要先了解一下RemoteInput。RemoteInput原先是在Android Wearable的API中使用的，通过Notification.Builder的setRemoteInput方法设置，可以让用户在手表的通知上进行语音回复。现在RemoteInput也能进行文字输入了，它将在这里发挥重要作用。

我们知道，Android L开始，通知可以通过添加action，在通知控件上添加额外的交互控件来提供额外的操作，这里也一样，RemoteInput是通过Notification.Action加入到Notification中的

### RemoteInput

``` java
RemoteInput remoteInput = new RemoteInput.Builder(KEY_REPLY)
        .setLabel(label)
        .build();
```

RemoteInput使用了建造者模式，RemoteInput.Builder的构造函数中需要提供一个String类型的resultKey，这个key将会在后面提取用户输入内内容的时候用到。setLabel()方法是设置用户点开输入框后的hint文字


----------
### NotificationCompat.Action

由于RemoteInput是v4 support包的，因此这里的通知是使用v4 support的NotificationCompat进行构建的

``` java
NotificationCompat.Action action = new NotificationCompat.Action.Builder(
                R.mipmap.ic_launcher,
                label,
                pendingIntent
        )
        .addRemoteInput(remoteInput)
        .build();
```

Builder构造器的三个参数分别是该action对应的icon资源（在这里好像没什么用），action的标题，对应的PendingIntent（用于启动处理消息回复）。
Builder调用addRemoteInput()方法设置RemoteInput

#### PendingIntent

我们这里的PendingIntent是用来启动负责消息回复的成员，它可以是：

 - Service或者BroadcastReceiver
 它们没有UI，适合后台操作，因此是直接回复通知的最佳选择
 - Activity
 具有UI ，会关闭通知栏，会要求用户解锁设备

因此对于Android N以下的系统，由于SDK不直接支持RemoteInput，而且你需要让用户知道程序有没有在进行回复，应该启动Activity。对于Android N或以上的系统，应该启动Service或BroadcastReceiver

```java
Intent sendIntent = new Intent(this, SendService.class);
PendingIntent pendingIntent = PendingIntent.getService(
        this,
        CODE_NEW_MESSAGE,
        sendIntent,
        PendingIntent.FLAG_ONE_SHOT
);
```


----------
### NotificationCompat.Builder

Notification.Builder的使用以及通知的发送就不用介绍了：

``` java
builder = new NotificationCompat.Builder(this)
        .setSmallIcon(R.mipmap.ic_launcher)
        .addAction(action)
        .setStyle(style);
NotificationManager manager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
manager.notify(ID_NEW_MESSAGE, builder.build());
```

Notification.Builder的setStyle方法可以给通知添加样式，API预置的样式有：

 - Notification.BigPictureStyle
 - Notification.BigTextStyle
 - Notification.DecoratedCustomViewStyle
 - Notification.InboxStyle
 - Notification.MediaStyle
 - Notification.MessagingStyle

其中MessagingStyle很适合在这种情境下使用

``` java
style = new NotificationCompat.MessagingStyle("我")
        .setConversationTitle("吃货日常")
        .addMessage("今晚吃点啥", System.currentTimeMillis(), "mahong");
```

其中构造器的参数是用户的显示名，setConversationTitle()是设置此次对话的标题，addMessage()是添加一条消息，参数分别是消息内容，时间戳以及对方的用户名（如果是用户自己发出的则设为null）


----------
## 处理回复

当用户进行了回复后，回复内容会被包装在一个Intent中，发送到action的PendingIntent指向的Service，BroadcastReceiver或Activity

取出Intent内容的方法不是通常的getExtra()，而是通过RemoteInput.getResultFromIntent()取出的

``` java
@Override
protected void onHandleIntent(Intent intent) {
    Bundle bundle = RemoteInput.getResultsFromIntent(intent);
    if (bundle != null) {
        CharSequence replyText = bundle.getCharSequence(ReceiveService.KEY_REPLY);

        Intent replyIntent = new Intent();
        replyIntent.setAction("com.example.mahong.test.SEND_OVER");
        replyIntent.putExtra(ReceiveService.KEY_REPLY, replyText);
        sendBroadcast(replyIntent);
    }
}
```

这里省略去了发送回复的逻辑

需要注意的是，从Bundle中取出文本时使用的键，是创建RemoteInput对象时指定的键


----------
## 更新通知

用户点击发送按钮回复后，发送按钮会变成一个progress小圆圈，如果你在上面那步就结束的话，这个progress小圆圈就会一直转个不停，这样的用户体验显然是不好的，因此需要在消息发送成功后更新通知

``` java
@Override
public void onReceive(Context context, Intent intent) {
    CharSequence sequence = intent.getCharSequenceExtra(KEY_REPLY);
    style.addMessage(sequence, System.currentTimeMillis(), null);

    NotificationManager manager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
    manager.notify(1, builder.build());
}
```

如果是使用MessagingStyle的话，用addMessage()加上用户的回复，然后重新build()一下，用相同的id再发一次通知就行了

对于其他的style，则需要使用setRemoteInputHistory()来添加回复


----------

整个流程大概就是这样，让我们来看看效果：

![](http://od654njdm.bkt.clouddn.com/test.gif)

