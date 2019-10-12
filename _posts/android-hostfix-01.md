---
title: Android热补丁技术-01-AndFix 
date: 2016-8-16 16:48:45
categories: Android
tags:
- Android
- 热补丁
- 热修复
---
<Excerpt in index | 首页摘要> 
> Android热补丁技术-01-AndFix 
>
<!-- more -->
<The rest of contents | 余下全文>  
  
###概述
在过去，我们如果版本中如果出现极其严重的bug。一般怎么做？  
  
修复Bug->打包->上传应用市场->然后通知用户更新->用户选择更新->完成Bug修复  
  
当然，这是最理想的状态，但是如果用户就是不想升级呢?  
  
那就让他一直用有bug的版本呗.当然是不可能的，难道就没有动态修复bug而且又不用用户重新下载APP的方法么？ 自然是有的。这门技术叫做<font color='red'>热补丁</font>.  
  
###Android下的实现总的来说一般有两种:  
  
1.修改函数指针.2.利用[Gradle打包规则](https://mp.weixin.qq.com/s?__biz=MzI1MTA1MzM2Nw==&mid=400118620&idx=1&sn=b4fdd5055731290eef12ad0d17f39d4a&scene=1&srcid=1106Imu9ZgwybID13e7y2nEi#wechat_redirect)  
  
这两种方式的代表作应该是:1:[AndFix(Alibaba)](https://github.com/alibaba/AndFix) 2.[Nuwa(女娲、国内大神)](https://github.com/jasonross/Nuwa)  
  
犹豫了很久，nuwa存在一些安全隐患，并且 作者本人貌似也没有什么继续更新的意思了.[nuwa详情](http://jiajixin.cn/2015/12/16/nvwa/)  
  
简单说一下Andfix吧.  

####导入  
两种方式，推荐第一种.  
1:下载官方源码，然后导入library  
2:直接在build.gradle文件中加入:compile 'com.alipay.euler:andfix:0.4.0@aar'  
  
####使用:
在Application中加入初始化代码：  
```java
	//推荐些一个单例把mPatchManager持久化保存
	mPatchManager = new PatchManager(context);
	mPatchManager.init(AppUtils.getAppName(context));
	mPatchManager.loadPatch();  
```
然后在Splash界面去和后台交互，获取当前的版本的补丁，然后下载下来(下载代码代码略。)  
  
然后下载完毕之后调用:  
```java
	mPatchManager.addPatch(downloadFile.getAbsolutePath());  
```
即可.  
  
###再说一下补丁的生成:
假设我们发布了1个versionName:1.0.0 versionCode:1的app  
  
里面存在一个极其严重的bug.  
```java
	public void sum(){
		int bug = 10 / 0;
	}  
```
bug产生了，我们需要修复它.  
  
首先找到bug代码,修改它:  
```java
	public void sum(){
		int bug = 10 / 1;
	}    
```
重新打包编译:  
  
然后利用andfix的补丁生成工具(andfix的github上面有下载地址)  
  
将老的apk和新的apk放在补丁生成工具的目录下:  
  
然后使用:  
```shell
apkpatch -f new.apk -t bug.apk -k qingxiang.keystory -p password1 -a anotherName -e password2  
```
new.apk:修复了bug的apk  
bug.apk:线上已经发布的存在bug的apk  
qignxiang.leystory:你的签名  
password1:签名密码  
anotherName:签名别名  
password2:别名密码  
  
然后会在当前工具存放的磁盘根目录下生成(或者d盘，因为我放在d盘的不太清楚)  
  
当然你可以利用 -o 指定输出目录.  
  
##重点
<font color='red' size='3'>
AndFix是个大坑，目前发现不兼容机型:魅族全系列,锤子OS,金立手机.  
并且锤子一加载补丁就闪退！！强烈不建议使用,如果特别需要这们技术可以看看:
</font>
[RocooFix](https://github.com/dodola/RocooFix)  
我也刚开始看，后面会专门写一篇博客介绍