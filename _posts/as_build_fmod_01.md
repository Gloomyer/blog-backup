---
title: AndroidStudio利用ndk-build编译fmod
date: 2017-6-29 14:41:04
categories: Android
tags: 
- ndk
- fmod
---
<Excerpt in index | 首页摘要> 
> AndroidStudio利用ndk-build编译fmod
> <!-- more -->
> <The rest of contents | 余下全文> 

##  前提  ##
fmod是一款声音方面的c层Library

最开始本来想利用cmake来调用的，后来发现坑太多。

还是老的ndk-build方式更好用。

ndk-build 加入环境变量这个环境我不讲了。

##  正戏  ##
首先，下载fmod的android版本。

去[fmod官网](www.fmod.org)下载

然后创建一个项目。

包名要用 "org.fmod.example" 因为他的demo里面用的这个 如果用自己的 你需要去重新生成头文件 挺麻烦的 干脆就用他的好了。

不要选中！ include C++ support

首先在你的项目的gradle.properties文件中加入

```xml
android.useDeprecatedNdk=true
```

然后打开下载好的fmod文件的 api->lowlevel->examples->java->org->fmod->example 把里面的MainActivity 替换到你项目里面

找到api->lowlevel->lib->fmod.jar 放入你的libs目录中去 同步一下

在你的model(一般是app)上右键->new->Folder->JniFolder finish

在你的main目录下生成了jni目录

将 fmod 的 api->lowlevel->inc 整个目录里面的所有内容拷贝至jni目录下


将 fmod 的 api->lowlevel->examples下的 


common.cpp common.h common_platform.cpp common_platform.h play_sound.cpp 这几个文件也拷贝至jni目录下

这几个文件实现了一个简单的声音播放功能,用来当做示例.

继续找到 api->lowlevel->lib->armeabi 下的两个so也拷贝至jni目录

在jni目录下创建Android.mk文件

内容如下:

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)
LOCAL_MODULE    := fmod
LOCAL_SRC_FILES := libfmod.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE    := fmodL
LOCAL_SRC_FILES := libfmodL.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE    := playsound
LOCAL_SRC_FILES := play_sound.cpp common.cpp common_platform.cpp
LOCAL_SHARED_LIBRARIES  := fmod fmodL
include $(BUILD_SHARED_LIBRARY)
```

创建Application.mk文件

```
APP_ABI := armeabi
APP_STL := gnustl_static
```

然后打开terminal窗口 进入到 modelName(app)/src/main/java目录下

输入 ndk-build

然后在main目录下创建jniLibs目录，讲main/libs目录下的文件拷贝至jniLbis目录下

打开MainActivity

拉到最底下 static代码块中内容全部注释

然后里面加入

```java
System.loadLibrary("playsound");
```

然后打开app里面的build.gradle文件

在android节点下加入

```
sourceSets.main {
   jni.srcDirs = []
}
```

然后在main目录下创建assets目录

然后在fmod的api->lowlevel->examples->media 目录下

找到 drumloop.wav jaguar.wav swish.wav 拷贝至assets目录下

然后就可以运行你的app啦！

##  源码下载  ##

[源码下载](http://gloomyer.com/upload/as_build_fmod_01.7z)