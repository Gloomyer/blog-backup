---
title: ZoomImageView 可以自由缩放的大图预览ImageView
date: 2016-7-5 12:39:20
categories: Android
tags:
- Android
- 自定义View
---
<Excerpt in index | 首页摘要> 
>ZoomImageView 可以自由缩放的大图预览ImageView  
>
<!-- more -->
<The rest of contents | 余下全文>  
  
公司项目用到了大图浏览。老版本用的是知名框架[PhotoView](https://github.com/chrisbanes/PhotoView)  
  
比较完美，但是有个很大的问题，公司APP中有漫画的成分。碰到这种情况就需要全图直接加载，方便阅读。  
  
头疼了一阵，最后决定自己写一个大图浏览框架。  
  
框架是利用ImageView的ScaleType来缩放和移动的，也解决了ViewPager滑动冲突的问题。  
  
具体怎么做的就不说了，直接去看源码吧。  
  
GitHub地址：  
[ZoomImageView](https://github.com/Gloomyer/ZoomImageView)