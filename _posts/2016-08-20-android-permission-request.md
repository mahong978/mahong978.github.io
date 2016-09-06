---
layout: post
title:  "Android M之权限请求"
date:   2016-08-20 16:55
categories: Android
tags: Android
---


* content
{:toc}

从Android Marshmallow开始，应用权限是在运行时获得的，而不是6.0之前的安装时获得，当然这是为用户的使用体验考虑，毕竟安装应用时一长串的权限说明看起来也是挺晕的（这是
对于在Google Play商店下载安装应用而言，安装未知来源APK对于6.0和pre6.0都会在安装时看到一长串的权限说明）

当然也并不是说对与pre6.0的版本就没有用，用户是有可能在安装应用后限制应用的权限的（虽然原生的权限管理是Android M才开始加入的，不过很多定制系统都有加入权限管理功能，因此这也是需要考虑的），所以运行时的权限检查是有必要的，因此Google也提供了相对应的support包

本篇将对Android Support Library提供的权限请求功能，进行相应的学习和研究







------------
## 权限分类
Android M对权限进行了分类。系统权限可以分为两种：一般的（normal）和危险的（dangerous）

### normal permissions
 那些可能需要访问数据和资源，但对用户的隐私的影响比较小的权限。如果应用已经在Android Manifest中声明了需要这些权限，系统会默认允许，而不会和用户交互。包含的权限有：
   
   - ACCESS_LOCATION_EXTRA_COMMANDS
   - ACCESS_NETWORK_STATE
   - ACCESS_NOTIFICATION_POLICY
   - ACCESS_WIFI_STATE
   - BLUETOOTH
   - BLUETOOTH_ADMIN
   - BROADCAST_STICKY
   - CHANGE_NETWORK_STATE
   - CHANGE_WIFI_MULTICAST_STATE
   - CHANGE_WIFI_STATE
   - DISABLE_KEYGUARD
   - EXPAND_STATUS_BAR
   - GET_PACKAGE_SIZE
   - INSTALL_SHORTCUT
   - INTERNET
   - KILL_BACKGROUND_PROCESSES
   - MODIFY_AUDIO_SETTINGS
   - NFC
   - READ_SYNC_SETTINGS
   - READ_SYNC_STATS
   - RECEIVE_BOOT_COMPLETED
   - REORDER_TASKS
   - REQUEST_IGNORE_BATTERY_OPTIMIZATIONS
   - REQUEST_INSTALL_PACKAGES
   - SET_ALARM
   - SET_TIME_ZONE
   - SET_WALLPAPER
   - SET_WALLPAPER_HINTS
   - TRANSMIT_IR
   - UNINSTALL_SHORTCUT
   - USE_FINGERPRINT
   - VIBRATE
   - WAKE_LOCK
   - WRITE_SYNC_SETTINGS


### dangerous permissions
可能会访问用户隐私数据，修改用户和其他应用的数据的权限。危险的权限需要运行时进行请求
 
#### 权限分组

任何一个危险权限都归属与一个权限分组。权限组有如下行为：
1. 当应用请求一个权限，且该应用还未有该权限所在的权限组中的其他的权限时，请求权限所显示的对话框将会对该权限组进行描述，而不是这个权限
2. 当应用请求一个权限，且该应用已经具有该权限所在的权限组中一个权限时，系统将会默认地允许该请求，而不会显式地让用户决定
  
这些分组可能会在未来发生改变，所以不要太依赖权限分组，需要用到的权限还是要一个一个去检查和请求

各个组别如下：

|组名|权限|
|:-------|:-------|
|CALENDAR|READ_CALENDAR<br/>WRITE_CALENDAR|
|CAMERA|CAMERA|
|CONTACTS|READ_CONTACTS<br/>WRITE_CONTACTS<br>GET_ACCOUNTS|
|LOCATION|ACCESS_FINE_LOCATION<br>ACCESS_COARSE_LOCATION|
|MICROPHONE|RECORD_AUDIO|
|PHONE|READ_PHONE_STATE<BR>CALL_PHONE<BR/>READ_CALL_LOG<BR/>WRITE_CALL_LOG<BR/>ADD_VOICEMAIL<BR>USE_SIP<BR/>PROCESS_OUTGOING_CALLS|
|SENSORS|BODY_SENSORS|
|SMS|SEND_SMS<BR/>RECEIVE_SMS<BR>READ_SMS<BR/>RECEIVE_WAP_PUSH<BR/>RECEIVE_MMS|
|STORGE|READ_EXTERNAL_STORAGE<BR>WRITE_EXTERNAL_STORAGE|
     
     


----------
## 检查权限
在进行涉及危险权限的操作前，需要先检查应用是否具有对应的权限。建议总是在操作前检查权限，因为用户有可能会在设置面板取消对应的权限
调用`ActivityCompat.checkSelfPermission()`进行检查：

```java
int isPermissionReady = ActivityCompat.checkSelfPermission(this, Manifest.permission.READ_CONTACTS);
```

需要提供的参数是当前的上下文和对应的权限。`android.Manifest.permission`中定义了各个权限对应的标志

如果已经拥有该权限，则会返回`PackageManager.PERMISSION_GRANTED`，否则返回`PackageManager.PERMISSION_DENIED`


----------
## 请求权限

### 解释为何需要这个权限

Android考虑到用户的体验，当用户拒绝了第一次的权限请求后，再次收到该权限请求时，对话框中会有一个“Don't ask again”的checkbox，用户勾选了这个CB后将不会再收到这个权限的请求。那么开发者该如何知道呢？

`ActivityCompat`有一个方法`shouldShowRequestPermissionRationale()`，字面意思是“是否应该显示所请求权限的理由”，表示你是否需要对用户说明这个功能是怎样的，为何需要这个权限。如果用户已经拒绝了之前的请求，该方法会返回true；当用户拒绝了之前的请求并勾选了Don't ask again”的checkbox，或者用户主动在权限管理中取消了该权限，则会返回false

### 请求权限

请求权限使用`ActivityCompat.requestPermissions()`方法，需要提供参数是上下文，权限标志数组，以及这次请求对应的请求码。

当然权限的请求必须是异步的，因为从显示对话框，再到用户对对话框做出选择都是需要时间的，`requestPermissions()`会立即返回，至于请求的结果则会在回掉函数中体现（下面讲）

结合上面的内容，一次请求应该这么写：

```java
private void readContacts() {
    if (Activity.checkSelfPermission(this, 
            Manifest.permission.READ_CONTACTS) !=
        PackageManager.PERMISSION_GRANTED) {
        
        //如果没有该权限，进行请求
        requestReadContactsPermission();
    } else {
    
        //进行联系人读取
    }
}

private void requestReadContactsPermission() {
    if (ActivityCompat.shouldShowRequestPermissionRationale(this,
               Manifest.permission.READ_CONTACTS)) {
               
        //进行相应的解释。要注意必须是异步的，不要阻塞主线程
        
    } else {
        //直接请求
        ActivityCompat.requestPermissions(thisActivity,
            new String[]{Manifest.permission.READ_CONTACTS},
            REQUEST_READ_CONTACTS};
    }
}
```

### 处理请求结果

由于权限请求是异步的，因此请求结果将会以回调的形式返回，ActivityCompat类中有一个回调接口`OnRequestPermissionsResultCallback`，因此请求代码所处的Activity需要实现这个接口

这个接口中只有一个方法：

```java
onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults)
```

参数依次是你的请求所设置的请求码，权限列表，以及每个权限对应的请求结果

具体实现可以这么写：

```java
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String permissions[], 
    @NonNull int[] grantResults) {
    switch (requestCode) {
        case REQUEST_READ_CONTACTS:
            if (grantResults.length > 0 && grantResults[0] 
                == PackageManager.PERMISSION_GRANTED) {
                
                //获得权限，继续对应的操作
                readContacts();
            } else {

                //未能获得权限，取消该操作但不要让应用退出
            }
            break;
        //...
    }
}
```

参数中的权限数组和结果数组是一一对应的。根据请求码判断是哪个请求，并对各个权限进行检查


----------
## 何时请求权限
根据Android Developer的介绍，权限请求的情况有两个因素：权限的重要性，以及用户对权限必要性的认识

讲权限的重要性分为重要的（critical）和次要的（secondary），用户的认识分为清除的（clear）和不清楚的（unclear）。对于一款相机应用来说，可以有如下四种情况：

1. 如果该权限对应用来说很重要，而且用户也很清楚，例如CAMERA权限。直接启动时请求权限，不需要多余地说明

2. 如果该权限对应用来说很重要，但用户不能马上认识到其必要，例如记录相片地理位置的功能。对于这种很吸引人的功能，或者是更新后加入的新特性，可以在用户教程（tutorial）界面时进行介绍，然后再请求权限

3. 如果该权限是比较次要的，但用户很清楚权限的必要，例如给相片加入语音。对于这种功能，当用户点击使用，直接在上下文种请求权限就行了

4. 如果该权限是比较次要的，而且用户不能马上认识到其必要，例如读取短信中的验证码。自动读取短信中的验证码是有用的，但用户有时并不能马上认识到它的便利，这样的话可以在上下文中适当地说明这个功能，当用户启动该功能时再请求权限

引用官方的说明图：

![](http://img.blog.csdn.net/20160820205456632)


----------
关于Android M的权限请求大概就是这么多。从官方的说明文档可以看出来，权限请求这个改动其实是充满了设计感的，无论是请求策略，还是请求代码，都体现了Android在用户体验，以及帮助开发者开发更优质的应用，所做出的努力。谷歌在优质化应用上所做的其实并不止是Material Design，很多小细节都在大大地改变Android应用在人们心中的印象。

另附：

[官方说明文档](https://developer.android.com/guide/topics/security/permissions.html#normal-dangerous)

[android-RuntimePermissions](https://github.com/googlesamples/android-RuntimePermissions)