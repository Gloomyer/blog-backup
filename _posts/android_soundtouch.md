---
title: 利用SoundTouch降低音调 实现变声(变男、变女)
date: 2018-01-26 15:58:54
categories: Android
tags: 
- ndk
- soundtouch
---
<Excerpt in index | 首页摘要> 
> 利用SoundTouch降低音调 实现变声(变男、变女)
> <!-- more -->
> <The rest of contents | 余下全文> 

##  写在前面 ##
soundtouch是一个c library 所以我们需要用到jni

你需要有jni的基础知识

本篇文章只介绍如何编译、使用它 

不做它的代码分析

你需要准备好ndk/sdk 并且配置好环境变量

##  前提  ##
之前有一篇文章介绍如何模仿qq魔声

[利用Fmod模仿QQ魔音效果](https://gloomyer.com/2017/06/30/as_imitationqq_beats_by_fmod/)

fmod 功能更为强大，但是同样的有些方面缺陷也很大。

fmod是一个音频播放处理框架

从名字上就应该能知道它是在播放的时候处理对音频做手脚。

其实如果变声只是在app内部流通  可以采用多端传递音频原生文件+变声参数

各端播放的时候做处理即可。

我们这边希望能够做到音频分享到微信直接播放，必须将音频文件传递给微信，而且微信播放音频的过程我们无法干预 所以fmod是不适于这个场景的。

soundtouch应势而出

soundtouch是一个音频处理库

soundtouch 的功能不是很强大 主要可以调整的参数只有2个：音速、音调

对我们来说实现上是足够了，因为声音变男、变女只需要做变男（降低音调） 变女（提高音调）就可以实现。

[SoundTouch官网](http://www.surina.net/soundtouch/)

## 下载、编译 源码  ##
写这篇文章的时候，soundtouch的最新版本是2.0

[soundtouch2.0源码官网下载地址](http://www.surina.net/soundtouch/soundtouch-2.0.0.zip)

下载下来它 并解压

源码需要修改2个地方才能保证编译通过

1:soundtouch根目录/source/Android-lib/jni/Application.mk

在里面加入这句话

```
APP_ALLOW_MISSING_DEPS := true
```

2:soundtouch根目录/source/SoundTouch/BPMDetect.cpp

打开搜索fabs (238行)

修改为abs

进入 soundtouch根目录/source/Android-lib/jni 目录

打开终端(windows在当前目录打开命令行)

输入ndk-build (需要下载并且配置ndk环境变量)

然后就成功的生成了我们需要的so文件 (在soundtouch根目录/source/Android-lib/libs 目录下)

打开我们的项目

将所有的so copy至我们的项目的jniLibs目录下(如果是配置so的导入路径 你配置的哪里粘贴到哪里去)

然后copy soundtouch根目录/source/Android-lib/src/net/surina/soundtouch/SoundTouch.java 到你的项目里面 注意 这个文件的包名不能变

如果需要该这个 那你还需要去修改jni里面的soundtouch-jni.cpp 然后重新buildso

## 使用  ##

变声,注意需要运行在子线程中，因为是耗时操作

```
SoundTouch st = new SoundTouch();
st.setTempo(params.tempo);
st.setPitchSemiTones(params.pitch);
int res = st.processFile(params.inFileName, params.outFileName);
if (res != 0) {
	String err = SoundTouch.getErrorString();//失败
}else{
	//成功
}
```

解释一下

tempo是音速，取值0.01 - 1 这个我们不用管1.0f即可

pitch 是音调 这个就是我们的重点了， 大于0 是变女生，小于0是变男声

具体的效果需要和产品去调整 

我们调整出来的是比较合适的值是 

男声:-10
女声:+10

inFileName : 要转换的文件全路径 必须为wav格式

outFileName:转换之后保存的文件全路径 必须为wav格式

因为soundtouch只支持wav的处理

我们多端传输使用的是aac，所以需要转换格式，这里推荐一个转换格式的库:

[音频格式转换库AndroidAudioConverter](https://github.com/adrielcafe/AndroidAudioConverter)

非常强大的一个工具库 支持的格式有 

```
AAC
MP3
M4A
WMA
WAV
FLAC
```

转换使用也非常的方便 傻瓜化的那种 这里就不介绍了