---
title: Android-ImageView获取显示的src图片
date: 2016-7-2 14:03:11
categories: Android
tags:
- Android
- ImageView
---
<Excerpt in index | 首页摘要> 
> Android-ImageView获取显示的src图片  
>
<!-- more -->
<The rest of contents | 余下全文>  
  
今天做分享功能，因为分享的时候要传递分享的图片所以需要获取一个利用Glide已经加载至ImageView上的图片  
  
  
代码：  
```java
	iv.setDrawingCacheEnabled(true);
    Bitmap bitmap = Bitmap.createScaledBitmap(iv.getDrawingCache(), 32, 32, true);
    intent.putExtra("bitmap", bitmap);
	iv.setDrawingCacheEnabled(false);  
```
获取图片之前必须调用setDrawingCacheEnable(true)，否则无法从ImageView对象上获取图像.  
  
获取之后必须调用setDrawingCacheEnabled(false)，否则下次获取还是原来的图像.  
  
还有要注意的，因为我的项目里面获取到的图像要利用Intent传递，而Intent对图像的大小限制是40kb.没错，40kb！！！！  
  
所以这里用的是Bitmap.createScaledBitmap();  
  
如果你是要用原图，不需要缩放，直接用Bitmap.createBitmap()即可.(<font color='red'>不可以直接用getDrawingCache()</font>)