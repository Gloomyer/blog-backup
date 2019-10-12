---
title: Glide加载图片导致图片变形的原因 
date: 2016-9-14 17:09:07
categories: Android
tags: 
- Glide
---
<Excerpt in index | 首页摘要> 
>Glide加载图片导致图片变形的原因
<!-- more -->
<The rest of contents | 余下全文> 
  
Glide在第一次加载图片。导致展位图变形的bug.  
  
再Glide加载的时候指定如下内容:
```java
 Glide.with(getActivity())
	.load(url)
	.placeholder(R.mipmap.area_img2)
	.dontAnimate()  //不适用加载动画
	.centerCrop()	//居中显示
	.into(iv);  
```