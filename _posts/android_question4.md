---
title: 最近碰到的几个问题记录
date: 2017-12-27 17:07:56
categories: Android
tags: 
- Android
---
<Excerpt in index | 首页摘要> 
> 播放视频的时候屏幕变暗
> 
> 关闭RecyclerView刷新动画
> 
> Realm的几个坑 
> 
> 播放视频的坑
> <!-- more -->
> <The rest of contents | 余下全文> 

###  播放视频的时候屏幕变暗 ###
因为用户设置的屏幕变暗时间很短，

app内涉及到播放视频的时候 有很多个短视频 用户超过时间不操作就会变暗

解决方案也很简单在activity启动的时候setContentView之前加入如下flag即可

```
getWindow().setFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON,WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
```

### 关闭RecyclerView刷新动画 ###
老版本的recyclerview 很麻烦 需要去copy DefaultItemAnimator 创建一个自定义的继承SimpleItemAnimator 然后修改里面的内容 有时候还会导致ui出现问题

新版本就简单多了，一句话解决问题

```
((SimpleItemAnimator) mActivity.mRecyclerView.getItemAnimator()).setSupportsChangeAnimations(false);
```

### Realm的几个坑 ###

greendao对于数据库的升级实在是有心无力。

决定引入Realm。

Realm具体的介绍可以去看[Realm官网](https://realm.io/)

说下realm有几个坑

---

Realm对象只能够在创建线程调用

例如，如果你在主线程创建了一个Realm对象 现在需要进行大量的数据插入、查询操作

因为是耗时操作 你想要放入子线程去操作。

那么你如果开了一个子线程直接操作主线程创建的Realm对象是会crash的的

正确的做法是利用Realm的异步任务操作，或者在新开的线程中去创建新的Realm对象。

---

查询数据的时候，类似如下:

```
RealmResults<FeedInfo> list = mRealm.where(FeedInfo.class).findAll();
list.sort("lastCreatedTime", Sort.ASCENDING);
```

这里查询到的数据是只能查询不能修改的。否则会抛异常!

也不能够对这个数据进行序列号操作 否则也会crash

因为这里返回的数据是代理对象 不是真正对象

如果需要对查询到的数据进行序列号、修改操作需要这么做

```
RealmResults<FeedInfo> list = mRealm.where(FeedInfo.class).findAll();
list.sort("lastCreatedTime", Sort.ASCENDING);
List<FeedInfo> result = mRealm.copyFromRealm(list);
```

这里返回的result才是可以自由修改和对其进行序列化操作。


### 播放视频 ###
新功能是短视频播放方面的功能

因为是在循环View中播放视频，所以选用了TextureView

关于 TextureView 可以去看这里 [TextureView官方介绍](https://developer.android.com/reference/android/view/TextureView.html) (需翻墙)

简单的说 如果你需要同时展示多个视频 推荐你使用TextureView

最开始播放器采用Android的MediaPlay，出现了如下几个问题。

1.isPlaying()并不准确.  
2.ios录制的视频getDuration()返回为0  
3.ios录制的视频无法做到边下边播。  

目前还没有找到原因。

我估测 可能是因为ios录制的mp4视频metadata方面的问题 但是直接在web浏览器中打开和ios播放是没问题的。很纳闷。

最后没什么好办法的情况下选择更换播放器为[ijkplayer](https://github.com/Bilibili/ijkplayer)

马上解决了如上的几个问题。

使用起来也非常简单

如果你只是播放传统的mp4/3gp等格式，那么直接通过gradle引入对应库即可

```
# required, enough for most devices.
compile 'tv.danmaku.ijk.media:ijkplayer-java:0.8.4'
compile 'tv.danmaku.ijk.media:ijkplayer-armv7a:0.8.4'

# Other ABIs: optional
compile 'tv.danmaku.ijk.media:ijkplayer-armv5:0.8.4'
//compile 'tv.danmaku.ijk.media:ijkplayer-arm64:0.8.4' 1
compile 'tv.danmaku.ijk.media:ijkplayer-x86:0.8.4'
//compile 'tv.danmaku.ijk.media:ijkplayer-x86_64:0.8.4' 2

# ExoPlayer as IMediaPlayer: optional, experimental
compile 'tv.danmaku.ijk.media:ijkplayer-exo:0.8.4'
```

1/2注释掉，因为这两个的minSdk是23，也不需要

然还在gradle配置下

```
ndk {
    ldLibs "log"
    abiFilters "armeabi", "armeabi-v7a", "x86"
}
```

然后之前的播放控制逻辑不变，new MediaPlayer的时候更换为new IjkMediaPlayer()即可

播放流程和MediaPlayer是一样的。

然后需要改动下   
setOnPreparedListener  
setOnVideoSizeChangedListener   
等几个回调

setOnVideoSizeChangedListener 这里会有5个参数（原来是三个）

老的是mp, mVideoWidth, mVideoHeight

新的是 mp, mVideoWidth, mVideoHeight, sar_num, sar_den 复制过去即可

