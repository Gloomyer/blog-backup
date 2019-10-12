---
title: Android几个目前不常见的问题
date: 2017-11-14 17:02:07
categories: Android
tags: 
- Android
---
<Excerpt in index | 首页摘要> 
>  Android几个目前不常见的问题
> 
>  Android推送目前的最佳解决方案
> <!-- more -->
> <The rest of contents | 余下全文> 

###  最近任务  ###
app中mainacitivity因为涉及到多次启动的问题所以设置启动模式为singleInstance

在小米机型上唤出最近任务的时候 一个app却显示有2个界面 一个main 一个之后打开的

这应该来说算是小米rom的bug?vivo上也出过这个问题

其实蛮简单的在manifest中注册activity的时候加上亲和度属性即可

```
 <activity
            android:name=".ui.main.MainActivity"
            android:launchMode="singleTask"
            android:screenOrientation="portrait"
            android:taskAffinity="com.feparty.tbh"
            android:theme="@style/NoSystemBarTheme" />
```

这个taskAffinity就是设置亲和度的熟悉

如果设置的不一样那么就会不显示在一起 如果设置一样 就会显示在一起

如果有一个app在最近任务中要求显示多个的需求一样就是定义这个属性设置成不同的即可

###  推送  ###
Android现在的推送基本就是个深坑

在app被用户从最近任务中清理掉之后，基本除非有rom渠道，否则根本收不到推送消息。

目前最好的提升成功率的方案就是多渠道推送。

即小米手机走小米推送、华为手机走华为推送、魅族手机走魅族推送 其他没有rom级渠道的走第三方供应商的推送

蛮麻烦的不过有好心人做了对应的整合

[OnePush 推送一个就够了](https://github.com/pengyuantao/OnePush)

说一下 采用上面的库的时候 在小米的id配置，如果你是Linux的系统 

他demo和文档上写的注册是这样的:

```
<!--小米推送静态注册-->
    <meta-data
        android:name="MI_PUSH_APP_ID"
        android:value="\ 2882303761517577233"/>

    <meta-data
        android:name="MI_PUSH_APP_KEY"
        android:value="\ 5701757717233"/>
```

但是如果你是Linux系统,(我是Ubuntu),那么这里不能填写空格 不然会出问题


改成类似如下:
```
<!--小米推送静态注册-->
    <meta-data
        android:name="MI_PUSH_APP_ID"
        android:value="\2882303761517577233"/>

    <meta-data
        android:name="MI_PUSH_APP_KEY"
        android:value="\5701757717233"/>
```

###  广播  ###
这是在接OnePush的时候发现，咨询作者之后才知道的问题

Android O(8.0) 之后 禁止静态注册跨进程通信广播.必须采用代码动态注册广播！

网上搜索的时候只有一个文章说7.0开始 如果是跨进程的广播，Action必须要在manifest中注册一下

[Android7.0的自定义广播接收需要权限了！！！](http://blog.csdn.net/u011043551/article/details/68962708)

这应该是指原生Android,小米开发板7.0的rom上不用提前注册这些action也能够正常接收广播.

如果你需要兼容8.0的手机 需要注意了。one push 依赖一个广播接受者来接受 各种形式的回调，你需要在app启动的时候 手动去注册一下那个广播接受者了.