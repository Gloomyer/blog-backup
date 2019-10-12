---
title: Android 编译IJKPlayer0.8.8 支持rtmp https
date: 2019-04-15 17:31:32
categories: Android
tags:
- Android
- JNI
- ijkplayer
---
<Excerpt in index | 首页摘要>
> Android 编译IJKPlayer0.8.8 支持rtmp https
>
<!-- more -->
<The rest of contents | 余下全文>  

## 概要

ijk官方提供的lib 是不支持https/rtmp 直播流的 播放

当然ijk是支持的 需要我们下载源码 修改参数 手动编译即可实现。

末尾提供懒人包下载

### 环境

ijk源码(0.8.8)

>git clone https://github.com/bilibili/ijkplayer.git

ndkr10e

> 官网写的是r10e 我用r19 失败了 但是网上有人说貌似13 14是成功的，看你自己了 省事直接用r10e
>
> [NDKr10e下载地址](<https://blog.csdn.net/shuzfan/article/details/52690554>)

系统

> 官网说是ubuntu，我用的mac成功了。windows没试过。

git

> 不用多说了吧



## 开始

准备好了就可以开始编译了

### 配置环境变量

ANDROID_NDK

> 指向ndk目录

ANDROID_SDK

> 指向sdk目录

没个系统环境变量配置有所不同 自己根据自己系统来就好

### 下载ijk源码

> git clone https://github.com/bilibili/ijkplayer.git

### 修改要支持的协议

~~~shell
cd ijkplayer
cd config
~~~

>module-default.sh   module-lite-hevc.sh module-lite.sh      module.sh

module-default.sh

> 较多的格式支持 编译之后文件体积过大（但是我没在里面找到rtmp??）

module-lite-hevc.sh

> 较少的格式支持 编译之后文件体积最小 不推荐使用

module-lite.sh

> 默认的格式支持 我是直接使用这个改了改

module.sh

> ijk编译使用哪个 这是一个链接文件指向module-lite.sh

~~~shell
vim module-lite.sh
~~~

搜索rtm 找到几个 如下

>export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-protocol=rtmp*"
>
>export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-protocol=rtmp"
>
>export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-protocol=rtmpt"

如果前面有disabled的 修改为enable即可

###初始化依赖文件

```shell
#回到ijk根目录
cd ..
#这是支持https
./init-android-openssl.sh
#这是Android平台使用
./init-android.sh
```



### 开始编译

```shell
cd android/contrib

#清除缓存
./compile-openssl.sh clean
./compile-ffmpeg.sh clean

#开始编译依赖库
./compile-openssl.sh all
./compile-ffmpeg.sh all

#回到ijk根目录/android/
cd ..

#开始编译 all表示全平台 如果只要某个平台 修改为某个平台即可
./compile-ijk.sh all
```



### 完成

生成好的文件在

> ijkplayer/android/ijkplayer

ijkplayer-java tools

这两个目录必须 其他的就是cpu架构支持目录 根据需要选择即可



## 懒人包下载

[ijk0.8.8 支持rtmp https 懒人包](https://pan.baidu.com/s/1aDNGNb283RlBG0Yjb9PwmQ)

密码:w9nc