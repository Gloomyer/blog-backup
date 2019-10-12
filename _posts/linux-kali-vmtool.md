---
title: Kali2.0安装VmwareTool
date: 2016-9-8 14:58:56
categories: Hack
tags:
- Linux
- Kali
- VmTool
---
<Excerpt in index | 首页摘要> 
> Kali2.0安装VmwareTool
>
<!-- more -->
<The rest of contents | 余下全文>  
  
杭州G20 峰会，托了个福放了七天假。本想好好浪荡几天，每天都在坐车 累到死~  
  
昨天还没上班 又已经回到杭州实在无聊，就想捣鼓点什么，先是写了个Android端偷取用户相册、联系人、短信上传至服务器的小钓鱼木马(android/server).  
  
又想起来上次下载好了Kali.那就想着用Vmware装上它看看吧.  
  
安装过程略过...   
  
安装完了自然要安装神器Vmtool.  
  
安装好后显示了Enjoy！--Vmware Team.  
```shell
	reboot.  
```
what???无效？  
  
好吧.通过谷歌爸爸知道了 是因为kali新版并不适用Vmware自带的Vmtool，而是提供了一个他们自己开发的openvmwaretool.  
  
那好吧,先去卸载已经安装上的,目录为 /etc/Vmwaretool (大小写忘记了)  
  
然后首先编辑下源:
```shell
	vim /etc/apt/source.list  
```
注释掉老的源,我使用的是  
```txt
	#中科大kali源
	deb http://mirrors.ustc.edu.cn/kali sana main non-free contrib
```
然后执行  
```shell
	apt-get update & apt-get upgrade
	apt-get dist -upgrade
	apt-get clean
	reboot //重启
	//继续执行
	apt-get install open-vm-tools-desktop  
```
然后.你以为这就完了？Fuck，又失败了。
  
这次又提示缺少一个依赖. 我自己去下载这个  
```shell
	apt-get install libatkmm-1.6-1  
```
OK,GG，又失败了.  
  
这回提示一个C++的库的版本不对.  
```shell
	apt-get remove xxx //忘记了
	apt-get install xxx-2.0 bate // 具体的忘记了没有记.当时虚拟机还不能复制  
	reboot  
```
然后再次执行  
```shell
	apt-get install open-vm-tools-desktop
	reboot  
```
这回终于装上了.真的是一波三折~