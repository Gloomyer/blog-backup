---
title: android 源码学习-2[运行编译出来的rom]
date: 2017-12-28 17:41:45
categories: AOSP
tags: 
- AOSP
---
<Excerpt in index | 首页摘要> 
> android 源码学习-2 运行编译出来的rom
> <!-- more -->
> <The rest of contents | 余下全文> 

### 引导 ####
在上一章的文章中，我已经成功的编译了Android8.1的源码，这次我打算将编译好的Android系统通过模拟器运行起来。

### 安装加速器 ###
Android模拟器的运行速度一直遭人诟病，所幸的是我们有加速器来提升我们的模拟器的运行速度，我们先来安装它。

```
sudo apt-get install qemu-kvm
egrep -c '(vmx|svm)' /proc/cpuinfo
```

第一句是安装kvm加速器，第二句是检查是否安装成功，我这里返回的是4，正常应该是返回非0即可以认为是安装成功

### 运行模拟器 ###

首先打开一个终端，然后进入根目录（记得这里的根目录说的是哪里么？）。

然后加载Android的环境变量！如果你配置了环境变量，将不再需要自己配置
```
source build/envsetup.sh
```

启动Android模拟器需要四个文件，它们分别是kernel-qemu、system.img、userdata.img和ramdisk.img，其中，前面一个是Linux内核镜像文件，而后面三个是Android系统镜像文件

```
echo $ANDROID_PRODUCT_OUT
```
这句话的返回路径就是后三者的存放路径，第一个后面我们再讲。

然后输入lunch
选择你编译的时候选择的架构
```
lunch
Which would you like? [aosp_arm-eng]  //我这里用的6 x86_64
```

最后启动模拟器
```
emulator
```

第一次启动可能比较慢，需要耐心等待一阵。

### 引用 ###
本文对于kvm的安装和模拟器的启动借鉴了[Android7.0源码编译运行指南](http://blog.csdn.net/HardReceiver/article/details/52650303)这篇文章