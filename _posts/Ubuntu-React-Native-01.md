---
title: Ubuntu16.10搭建React-Native开发环境
date: 2017-01-21 14:32:17
categories: Android
tags:
- Ubuntu
- React-Native
---
<Excerpt in index | 首页摘要> 
> Ubuntu16.10搭建React-Native开发环境
>
> <!-- more -->
> <The rest of contents | 余下全文>  



### 概述 ###

项目就在等时间上线了，乘着微信小程序的风，打算看看React-Native(RN)开发。

首先搭建一个基本的开发环境，看了下rn对windows的支持不是那么友好。

于是用虚拟机装了个Ubuntu来装开发环境



####  步骤  ####

我的目标是利用rn来开发android。

依赖的环境

androidSDK

build-tools-version == 23.0.1 (这个版本的一定要下载 其他无所谓)

nodejs



androidSdk安装太过于简单就不详细叙述了

重点是，安装完成要配置下环境变量

```shell
sudo apt-get install vim  //Ubuntu16.10默认不带vim 我擦
vim ~/.bashrc 
//打开，然后在文件末尾添加 
////根据实际情况来
export ANDROID_HOME=/home/gloomy/Developer/AndroidSdk 
export PATH=$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools
//保存退出
source ~/.bashrc //更新一下
//调用 adb android 两个命令来保证环境变量的添加正确
```

安装nodejs

如果直接使用sudo apt-get install nodejs 安装的是4.x版本有问题！

首先去官网下载nodejs 我下载的是6.9.x版本的

解压

放到一个你常用的配置目录比如:

```shell
mv node /home/lib/ //这是我爱放的目录
//然后添加进入系统命令
sudo ln -s /home/lib/node/bin/node /usr/local/bin/node
//注意下npm 你可以先用
sudo ln -s /home/lib/node/bin/npm /usr/local/bin/npm
//如果不行!(我就不行 我也不知道为什么)
//进入 /home/lib/node/bin 目录
//然后ls -l
//可以看到这里的npm也是个连接.指向的是../lib/node_modules/npm/bin/npm-cli.js
//那么，就可以这么做
//删除前面创建的npm
sudo rm /usr/local/bin/npm
//增加新的，直接指向npm的真实地址
sudo ln -s /home/lib/node/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm
//然后基本就没问题了
//运行下 nodejs -v 与 npm 来保证安装的正确性
```

安装完成了nodejs.

现在就可以通过npm来添加react-native了

```shell
//首先下载
npm install -g tarn react-native-cli
//然后 尝试一下 运行 react-native 如果不行！
//那么和npm的添加一个方式
sudo rm /usr/local/bin/react-native
sudo ln -s /home/lib/node/lib/node_modules/react-native-cli/index.js /usr/local/bin/react-native
//然后运行一下react-native 如果正常运行，打印一些帮助信息那么就说明成功了.
```



然后就是创建一个demo项目跑起来了！

```shell
cd ~
mkdir react-native
cd react-native
react-native init demo1 //第一次会下载很多东西有点慢..
cd demo1
//注意底下这句话
//执行之前开个终端 调用 adb devices
//保证返回的设备只有一个！，无论是真机还是模拟器 始终返回一个设备！！
react-native run-android //运行android项目到设备上,rn使用的gradle是2.4,如果没有这里会下载。。超慢..
```





恩，这就是在Ubuntu上搭建RN开发环境.

顺带一提，如果你的x86架构模拟器在虚拟机中无法打开.

首先，保证你的物理机bios开启了虚拟化设置.

然后.

关闭虚拟机，打开当前虚拟机设置

硬件(默认选项卡)->处理器->启用 "虚拟化 Intel VT-x/EPT 或 AMD-V/RVI(V)" 这个选项，然后重新打开。

再开机，然后启动x86模拟器启动应该就可以了.