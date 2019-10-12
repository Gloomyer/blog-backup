---
title: android 源码学习-1[下载、编译]
date: 2017-12-28 17:41:45
categories: AOSP
tags: 
- AOSP
---
<Excerpt in index | 首页摘要> 
> android 源码学习-1 下载、编译
> <!-- more -->
> <The rest of contents | 余下全文> 

### 引导 ####
在日常的开发中，越来越觉得了解源码的重要性。

因为很多功能如果对源码不够了解，出现了bug很难定位，只能去寻找替代方案。

但是很多情况下替换方案是很耗时的，并且未必能有什么好的替换方案。

如果对源码了解程度足够，那么在开发中也很容易去避免这方面的情况。

而且本人一直也对Android源码很感兴趣。

而且始终觉得一个Android开发者，连Android的源码都没编译过，哪里有好意思说自己是一个Android Developer。

综上所述，决定自己从编译Android开始，深入学习Android源码。

### 借鉴 ###
Android的源码学习注定是一个漫长、枯燥、繁琐的过程。

也需要对方方面面很多地方的了解。

我将借鉴

[老罗的Android之旅](http://0xcc0xcd.com/p/books/978-7-121-18108-5/index.php?page=4)

[AndroidOpenSourceProject](https://source.android.com/)

来一步一步的学习。

如果后续还借鉴到了别的地方，将会在文章底部附上借鉴的网址链接。

[老罗的Android之旅](http://0xcc0xcd.com/p/books/978-7-121-18108-5/index.php?page=4)是基于Android2.3.1的学习。

Android目前最新版本是Android8.1，本人也将基于Android8.1来学习Android源码。

### 准备 ###
Ubuntu16.04 64bit 电脑一台(可以是虚拟机) 12GB 内存 以上 200GB 可用空间以上 (如果你是虚拟机 分配磁盘的时候最好250GB以上 30gb的系统空间)

android 8.1 是基于openjdk1.8的，如果使用别的jdk会无法编译！

aosp(AndroidOpenSourceProject的简写 后续将直接用aosp代替)的很多内容都是基于google的，翻墙下载太过于繁琐，所以打算采用国内的镜像源。

一般有两个、[中科大](https://mirrors.ustc.edu.cn/)、[清华](https://mirrors.tuna.tsinghua.edu.cn/) 我这里采用清华的镜像源。

### Android的源码下载、编译 ###

#### 下载 ####
其实下载步骤在[清华](https://mirrors.tuna.tsinghua.edu.cn/) 的aosp里面说的很详细了，我这里就大致总结一下.

首先aosp的git方式管理的，但是为了方便我们下载编译，google提供了一个叫做repo的工具，它是一个对于git命令整合的工具。需要下载源码之前我们先需要下载repo [repo是什么](http://blog.csdn.net/qugename/article/details/57463951)

第一步，下载repo
```
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo
chmod +x repo
```

然后将repo 放到/usr/bin目录下去 方便我们直接调用
```
sudo mv repo /usr/bin/
```

第二部，下载Android源码

我们在home目录下创建WORKING_DIRECTORY的目录 把源码下载到这里去 并且以后也基本都是在这个目录下进行操作

```
cd ~
mkdir WORKING_DIRECTORY
cd WORKING_DIRECTORY
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-8.1.0_r2
```

repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest

是下载Android的所有代码从android-1.6_r1.2～ android-8.1.0_r2的所有源码

我们没必要下载全部 硬盘也不够，只需要下载我们需要的即可，

-b 参数就是指定下载的版本，我们这里采用android-8.1.0_r2 

具体的版本列表可以去看这里:[Android版本列表(需要翻墙)](https://source.android.com/setup/build-numbers#source-code-tags-and-builds)

init执行完成我们就可以开始下载源码了。

<font color='red'>注意:repo 会在下载过程中主动去尝试google官方的下载地址所以每次下载之前我们需要修改一下环境变量，当然你也可以直接配置到系统的环境变量里面去</font>

```
//设置repo的baseurl为清华的
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'
//开始下载代码
repo sync
```

下载代码是一件非常耗时的操作，Android8.1的源码我这里下载下来之后统计一共是57gb(下载了2天) 

中间难免会网络不畅、断网的情况出现

只要init成功之后 每次只需要执行 repo sync 即可，它是支持断点续传的。

但是如果repo的baseurl你没有配置到系统的环境变量里面去

那么记得 每次执行repo sync之前 执行一次

```
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'
```

#### 编译 ####
AOSP官网推荐使用的是ubuntu14.04 + openjdk1.8 

我这里采用的是ubuntu16.04 + openjdk1.8 目前看来也是正常没有出现问题的

编译之前我们来配置一些依赖的环境

首先安装必要的openjdk1.8 （每个Android版本依赖的jdk都有所区别 老罗的Android之旅中的2.1.3依赖的是oracleJdk1.6） 具体的可以去aosp官网查询

在执行第一步之前，我将ubuntu的源全部换成了清华的ubuntu源 提高apt速度

具体的操作可以去[清华ubuntu源帮助文档](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)中查询如何更换 

第一步，安装openjdk1.8
```
//删除电脑上之前的openjdk
sudo apt-get purge openjdk*
sudo add-apt-repository ppa:openjdk-r/ppa 
sudo apt-get update
sudo apt-get install openjdk-8-jdk
```
安装完成务必重启电脑，网上查询别人说重启终端即可，但是我这里单纯重启终端依然提示不支持的jdk版本

第二步，安装依赖环境

aosp是一个非常庞大的项目，所以来的开源工具也非常的多。

官方文档说的下载步骤走下来我发现是无法通过编译的 可能是官方推荐14.04 我用的16.04的关系 如果你也是使用16.04 请按照我这里的顺序执行一遍即可

按照顺序一步一步执行 下载即可。
```
sudo apt-get install bison g++-multilib git gperf libxml2-utils make python-networkx zlib1g-dev:i386 zip
sudo apt-get install gcc
```
依赖环境配置完成，那么我们可以考试尝试编译一次Android源码了。

首先进入最开始创建的WORKING_DIRECTORY目录，以后如果没有特殊情况我将会说'根目录'

配置jdk 栈大小，如果不配置很容易oom

```
export JACK_SERVER_VM_ARGUMENTS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx8192m"
out/host/linux-x86/bin/jack-admin kill-server
out/host/linux-x86/bin/jack-admin start-server
```

将编译需要的环境变量导入进系统

<font color='red'>你可以配置进系统的环境变量 如果没有配置以后每次开新的终端打算编译Android源码都需要执行一遍 后续将忽略这个步骤</font>
```
source build/envsetup.sh
```

如果你只打算便以某一个平台版本的
```
//选择构建版本
lunch
.....
.....
Which would you like? [aosp_arm-eng] //根据返回的信息输入你想要编译版本开头的数字回车即可 我这里使用的是x86_64，第6个 x86在linux上有加速器 模拟器的速度会提升很多 如果直接采用arm架构，模拟器的速度将卡的难以接受。
```

然后输入编译命令 后面的j8是定义的 -j[数字]的形式  这个数字一般为你电脑内核数量*2 我这里虚拟机是4核心单线程的cpu，所以采用的j8
```
make -j8
```
然后就开始执行编译了，这是一个非常耗时的操作。目前我跑了3个小时，依然没有完成。

这次就暂时到这里，下次会写如何把编译出来的rom通过模拟器跑起来