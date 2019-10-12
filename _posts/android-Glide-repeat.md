---
title: Glide加载本地图片重复问题
date: 2016-8-19 18:47:02
categories: Android
tags:
- Android
- Glide
---
<Excerpt in index | 首页摘要> 
> Glide加载本地图片重复问题  
>
<!-- more -->
<The rest of contents | 余下全文>  
  

项目中有个一个页面,上半部分是一个16:9的ImageView。下半部分是文字描述的说明，主要在上半部分。  
  
图片选择是调用系统相册，然后返回一张图片，再调用系统的裁剪模块，然后返回一张图片，返回的这张图片再利用Glide加载到上面去.(Glide支持File,Http,Https,content等方式解析图片,不得不说实在强大.)  
  
因为图片裁剪时调用系统的然后指定的一个sd卡的路径去存放裁切号的图片.  
  
因为这个裁切的图片是临时的，所有每次裁切采用的是一个全文件名.所以出现了一个Bug:  
  
选择图片->裁剪->返回图片(正常显示)->继续选择图片->裁剪->显示上一次的图片.(上传是正常的)  
  
考虑了一下，应该主要是Glide缓存策略导致的.  
  
我们在调用Glide的时候可以选择不使用任何缓存策略，每次加载都从磁盘上读取新的。  
  
方法如下:  
```java
	Glide.with(this)
		.load(imageBgPath)
		.skipMemoryCache(true) //跳过内存缓存
		.diskCacheStrategy(DiskCacheStrategy.NONE) //不使用磁盘缓存(?貌似即使是本地磁盘，Glide也会在磁盘利用自己的方式缓存一份)
		.into(editSerialIvBg);  
```