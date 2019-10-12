---
title: 开发中几个不常见的问题
date: 2017-11-19 20:43:36
categories: Android
tags: 
- Android
---
<Excerpt in index | 首页摘要> 
>  开发中几个不常见的问题
> 
> Android O (8.0) 发不了通知的问题
> <!-- more -->
> <The rest of contents | 余下全文> 

###  不可思议的oom  ###
说来也是有趣 昨天看了篇文章[Android不可思的oom](http://www.jianshu.com/p/e574f0ffdb42)

今天我就遇见了。

简直无敌

这个oom报错的信息如下:
```
java.lang.OutOfMemoryError

Could not allocate JNI Env

java.lang.Thread.nativeCreate(Native Method)
```
java.lang.Thread.nativeCreate(Native Method)

就是指调用了native的创建线程方法

简单的说，就是 new Thread().start() 报了oom！

博客里面写的很清楚了我就简单说一下

一般有几种情况导致的开线程oom

```
文件描述符(fd)数目超限，即proc/pid/fd下文件数目突破/proc/pid/limits中的限制。

线程数超限

内存不够了
```

解决方案:
比较有效的解决方案是app内一切需要开线程的操作 全部采用一个集中管理的线程池去避免.

###  JDK9的问题  ###
这个问题不属于Android开发懒得单独抽出来

我这里用的jdk直接用的最新的

as(linux3.0是正式版) 所以一直没有出什么问题

今天看netty的东西 想写个小demo

装了最新的idea出问题了。
```
Could not determine java version from '9.0.1'.
```
一直出这个错误

百度是shit，什么都查不出来 还是google给力

[解决方案](https://stackoverflow.com/questions/46867399/react-native-error-could-not-determine-java-version-from-9-0-1)

很简单 1.9的jdk需要搭配4.3的gradle 才可以正常使用 需要升级你的gradle

###   Android O (8.0) 发不了通知的问题  ###
如果使用最新的NotificationCompat发通知会惊奇的发现一件事。

new NotificationCompat.Build(BaseApp.getAppContext(), CHANNEL_ID);

多了一个参数，这个参数怎么填写呢？

如果你的targetSdk 是26 以下！ 那么直接填写null即可。

如果是26，那么恭喜你你发现你通知怎么发都发不出来了！

如果是仔细看logcat 就会发现打印了这么讲句话:
```
1: Use of stream types is deprecated for operations other than volume control

2: See the documentation of setSound() for what to use instead with android.media.AudioAttributes to qualify your playback use case
```

然后通过google发现了解决方案

[AndroidO发不了通知怎么办](https://stackoverflow.com/questions/45711925/failed-to-post-notification-on-channel-null-target-api-is-26)

简单的说一下，AndroidＯ在通知方面引入了Ｃｈａｎｎｅｌ的概念.

每种渠道(Channel)的通知展示根据你的设置有所区别。

具体的请自行ｇｏｏｇｌｅ

简单的解决方案如下

创建一个NotifyChannel
```
String CHANNEL_ID = "100";
if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.O) {
    //适配8.0使用通知 需要创建channel
    NotificationChannel mChannel = new NotificationChannel(CHANNEL_ID, AppUtils.getAppName(), NotificationManager.IMPORTANCE_DEFAULT);
    mChannel.enableLights(true);
    mChannel.setLightColor(Color.RED);
    mNotifyManager.createNotificationChannel(mChannel);
}
```

然后创建通知的时候代码如下:
```
 NotificationCompat.Builder builder = new NotificationCompat.Builder(BaseApp.getAppContext(), CHANNEL_ID)
        //设置图标
        .setSmallIcon(R.mipmap.ic_launcher)
        .setBadgeIconType(R.mipmap.ic_launcher)
        //设置通知标题
        .setContentTitle(AppUtils.getAppName())
        //设置通知内容
        .setContentText(message.get("content"))
        //设置ticker
        .setTicker(message.get("content"));

final Notification notification = builder.build();
notification.tickerText = message.get("content");

notification.defaults = Notification.DEFAULT_LIGHTS | Notification.DEFAULT_VIBRATE;

mNotifyManager.notify(getNotifyId(), notification);
```

现在就可以愉快的发通知啦~