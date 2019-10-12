---
title: RipplingView 涟漪动画
date: 2016-7-7 20:32:10
categories: Android
tags:
- Android
- 自定义View
---
<Excerpt in index | 首页摘要> 
>RipplingView 涟漪动画  
>
<!-- more -->
<The rest of contents | 余下全文>  
  
项目里面要做这个效果：  
  
![DEMOIMG](http://gloomyer.com/img/img/rippling_demo.gif)
  
模拟器录制的有点卡。  
  
简单说下思路：  
  
继承自View  
  
充血onDraw()方法  
  
画两个圆，一个不停地变大，一个不停地变小  
  
利用画笔的属性Style/Xfermode两个属性来实现透明  
  
具体的就不细说了，上传至github了，有兴趣的自己去看源码吧。  
  
[RipplingView](https://github.com/Gloomyer/RipplingView)