---
title: Android RecyclerView 嵌套RecycleView 导致内部RecyclerView没有完全显示
date: 2016-6-23 17:40:22
categories: Android
tags:
- Android
- 自定义View
---
<Excerpt in index | 首页摘要> 
RecycleView（LinearLayoutManager）  
嵌套RecycleView (GridLayoutManager)  
导致内部RecyclerView显示不完整解决方案
<!-- more -->
<The rest of contents | 余下全文>  
简单点说，外层是一个LinearLayoutManager的RecyclerView  
  
用于展示类似于动态的条目  
  
个别动态内部有九宫格的图预览。所以决定采用这种类型的item之间再嵌套一个RecyclerView（GridLayoutManager）  
  
出现了问题，两行/一行图片展示的时候是正常的。但是如果有三行，第三行就会显示一半。  
  
解决方式：  
在填充该item布局的时候viewgroup填写null  
  
原因：填写viewgroup的时候会利用viewgroup的根节点参数去填充布局，不填写则会默认用空的参数去填充，这样才能展示完整。  
  
如果以上方式还是显示不全，那就在createViewHolder（）的时候去直接计算并指定内部RecyclerView的Height。  
  
PS：  
在自定义的布局中，重写  
onTouchEvent（）  
onInterceptHoverEvent（）  
直接返回false，会一定的提升页面流畅度  
（减少计算事件分发代码计算量）