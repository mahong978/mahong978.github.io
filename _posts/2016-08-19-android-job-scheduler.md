---
layout: post
title:  "JobScheduler学习"
date:   2016-08-19 03:20
categories: Android
tags: Android
---


* content
{:toc}


后台任务是应用开发中常见的问题。

最简单的做法就是直接开一个Thread，用Handler通信即可，但是所开的线程和活动是没有关系的，一旦应用被杀死，就和之前所创建的线程失去了联系，就算活动再次启动，此时启动的线程并不是之前的线程。

当然，这个问题可以用Service解决，那么如果用户把服务也杀了呢？你可能会想用AlarmManager，周期性地进行唤醒，可是用户关机了，AlarmManager也就没了。当然这还没到“穷途末路”，用BroadcastReceiver就可以直接解决服务自启的所有问题。

但是让我们换个角度来看，打开管理器查看当前活动的服务就能看到，这些一直在活动的后台往往是导致手机耗电飞快的原因。当然这种问题是需要开发者的自觉的，如果让开发者来考虑省电，这将会是开发者的一大烦恼，比如说希望连接WiFi时才进行，插入充电器才进行，将会使代码大大增加；另一方面，万一此时没有网络可用呢？还得写一个BroadcastReceiver来监听网络情况。超级麻烦的不是吗？

Android L开始引入了一个新的API：JobScheduler。正如其名，它允许将任务调度直接交给系统，并且你所需要的许多约束条件，如周期调度，延迟调度，网络连接，电源插入，还有Android L引入的空闲模式（虽然现在还挺鸡肋），都可以快捷地进行设置。现在就开始对JobScheduler的学习






----------
## API成员

### JobInfo

代表一个任务，使用建造者模式进行制造，然后传递给JobScheduler进行调度管理。需要注意的是JobInfo类中的几个常量，建造时需要用到，至于作用会在后面的set方法中介绍：

 - BACKOFF_POLICY_EXPONENTIAL
 - BACKOFF_POLICY_LINEAR
 - DEFAULT_INITIAL_BACKOFF_MILLIS
 - MAX_BACKOFF_DELAY_MILLIS
 - NETWORK_TYPE_ANY
 - NETWORK_TYPE_NONE
 - NETWORK_TYPE_NOT_ROAMING
 - NETWORK_TYPE_UNMETERED
 
#### JobInfo.Builder
JobInfo的建造者，其构造器为：

```java
JobInfo.Builder(int jobId, ComponentName jobService)
```
   
第一个参数为该任务的标识符，该标识符在相同的uid的所有客户端中必须是唯一的（官方文档是这么写的，我的理解是在该设备上必须是唯一的）。为了保证在应用升级后也是稳定的，因此建议不要基于资源id进行设置

第二个参数是你希望用来处理该任务的服务对应的ComponentName，用来启动该服务
		
接下来是用来建造JobInfo的各个参数对应的set方法：

 - setBackoffCriteria(long initialBackoffMillis, int backoffPolicy)

   设置退避/重试策略。类似网络原理中的冲突退避，当一个任务的调度失败时需要重试，所采取的策略。

   第一个参数时第一次尝试重试的等待间隔，单位为毫秒，预设的参数有：

   - DEFAULT_INITIAL_BACKOFF_MILLIS

     30000

   - MAX_BACKOFF_DELAY_MILLIS

     18000000
     
   第二个参数是对应的退避策略，预设的参数有：

   - BACKOFF_POLICY_EXPONENTIAL

     二进制退避。等待间隔呈指数增长

   - BACKOFF_POLICY_LINEAR

 - setMinimumLatency(long minLatencyMillis)

   设置任务执行延迟的时长

 - setOverrideDeadline(long maxExecutionDelayMillis)
 
   设置任务的最大延迟时长。一旦达到该时间，无论条件是否满足，任务都会执行
  
   由于设置延迟对周期性任务来说没有意义，因此在build()时会抛出IllegalArgumentException异常

 - setPeriodic (long intervalMillis)

   设置周期。可以保证在每个间隔之间任务最多只执行一次
  
 - setPeriodic (long intervalMillis, long flexMillis)

   在周期末的一个flex长度的窗口，任务都有可能被执行
  
 - setPersisted (boolean isPersisted)

   设置设备重启后，这个任务是否还保留。需要RECEIVE_BOOT_COMPLETED权限
    
 - setRequiredNetworkType (int networkType)

   设置要求的网络，只有接入给定类型的网络才能执行，而且是必须要接入网络才能执行。预置的参数有：

 - NETWORK_TYPE_NONE

   默认值。表示与网络状态无关

 - NETWORK_TYPE_ANY

   必须连接网络

 - NETWORK_TYPE_NOT_ROAMING

   必须连接非漫游的网络

 - NETWORK_TYPE_UNMETERED
   
   必须连接非计费的网络

 - setRequiresCharging (boolean requiresCharging)

   设置是否需要充电器接入。默认false

 - setRequiresDeviceIdle (boolean requiresDeviceIdle)

   设置是否需要设备处于空闲状态。默认false。空闲状态指设置已经有一段时间没有被使用

 - addTriggerContentUri (JobInfo.TriggerContentUri uri)

   添加一个TriggerContentUri，该Uri将利用ContentObserver来监控一个Content Uri，当且仅当其发生变化时将触发任务的执行

   这个功能不能和setPeriodic(long)或者setPersisted(boolean)一起使用， 因为这样是没有意义的，否则在build()时会抛出IllegalArgumentException异常

   为了持续监控content的变化，你需要在最近的任务触发后再调度一个新的任务

 - setTriggerContentMaxDelay (long durationMs)

   设置从content变化到任务被执行，中间的最大延迟

 - setTriggerContentUpdateDelay (long durationMs)

   设置从content变化到任务被执行中间的延迟。如果在延迟期间content发生了变化，延迟会重新计算

 - build()

   创建对应的JobInfo
  
 - JobInfo.TriggerContentUri

     保存了一项任务触发绑定的content uri信息

     其构造器为：
	 
	
     ```
     JobInfo.TriggerContentUri(Uri uri, int flags)
     ```
	
	   第一个参数是希望监控的content的Uri，第二个参数是标识符


--------------
### JobParameters
当任务被调度，交由应用处理时提供的对象，包含了该任务的信息。无法自己创建JobParameter的实例

由于是用来提供任务信息的，所以基本都是get方法：

 - PersistableBundle getExtras()
 - int getJobId()
 - String getTriggeredContentAuthorities()

   获得触发该任务的content authorities

 - Uri[] getTriggeredContentUris()
 - boolean isOverrideDeadlineExpired()

   判断该调度是否因为达到deadline了


----------
### JobScheduler

负责调度任务。一般调用schedule(JobInfo)方法将任务加入到调度队列中。JobScheduler无法自己创建，因为是系统级的服务，所以用Context.getSystemService(Context.JOB_SCHEDULER_SERVICE)获得其实例

 - int schedule(JobInfo)

   将任务加入到调度队列中。将会返回一个结果
   
   - RESULT_SUCCESS

     加入成功

   - RESULT_FAILURE

     不合法的参数将会导致失败，有可能是该任务的run-time太短（不是很懂），或者其指定的JobService无法解析
	
 - cancel(int jobId)

   取消对应ID的任务

 - cancelAll()

   取消由这个包注册的所有任务

 - List< JobInfo > getAllPendingJobs()

   获得这个包注册的正在等待的任务

 - JobInfo getPendingJob(int jobId)

   获得指定的由该包注册的正在等待的任务



--------------
### JobService

JobScheduler的回调入口。

由于需要应用来完成任务的执行，因此需要继承该类，重载其onStartJob(JobParameter)方法，进行任务的执行

建议使用Handler来完成任务执行的逻辑，以防阻塞未来的任务

在Manifest注册该Service时，要注意添加android.permission.BIND_JOB_SERVICE权限，否则会被系统忽略

 - void jobFinished (JobParameters params, boolean needsReschedule)

   当完成任务的执行时，调用该方法通知JobManager。该方法可以在任何线程调用

   第一个参数是该任务对应的JobParameter，你会在onStartJob方法获得

   第二个参数表示是否需要重新调度。退避机制会产生作用，也就是重调度的任务加入到队列中后，将会等待一段时间才能获得调度。在设备空闲模式下，退避机制不会产生作用

 - boolean onStartJob (JobParameters params)

   JobService将会在这个回调方法获得可执行的任务，如果该任务不需要额外的执行，可以立即返回false。否则需要在单独的线程中执行（使用Handler），并且返回true，在任务执行完后调用jobFinished()方法进行通知

 - boolean onStopJob (JobParameters params)

   当你主动通知任务执行完毕（jobFinished）之前，系统可能会要求你停止任务，这时将会调用onStopJob方法

   当该任务的需求不再满足时将发生这种状况，必须对此做出反应，否则应用可能会出现行为异常。一种立即引起的影响就是系统可能会将你的wakelock释放

   返回true表示你希望对该任务重新进行调度，同样需要遵守退避策略；返回false表示你希望放弃该任务


----------

关于JobScheduler API的学习大概就是这么多了，可以看到Google从Android L开始就很关注电池续航的问题，包括M的doze mode也是一样，估计N也会引入更好的省电策略。看到网上关于这块的中文资料都不是挺全面，于是就自己尝试对整个API进行学习，感觉还行吧，就酱。

另附： 

[官方Sample](https://github.com/googlesamples/android-JobScheduler)

[JobScheduler官方文档](https://developer.android.com/reference/android/app/job/package-summary.html)