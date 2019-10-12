---
title: RecycleView之OnScrollListener的一个坑
date: 2016-6-27 17:44:51
categories: Android
tags:
- Android
- 开发深坑
---
<Excerpt in index | 首页摘要> 
>Android RecycleView之OnScrollListener的一个坑  
>
<!-- more -->
<The rest of contents | 余下全文>  
  
今天发现一个坑  
  
模块中有一个界面是上半部分都是滚动的RecyclerView，底部有一个EditText  
  
如果底部的EditText获取焦点并且输入法打开了  
  
那么就需要在RecyclerView滚动的时候去关闭输入法  
  
通过RecyclerView.addOnScrollListener设置了监听并实现了关闭逻辑。  
  
Bug出现，点击EditText，输入法根本就不显示了。  
  
通过日志发现，弹出输入法的时候会触发RecyclerView的滚动监听  
  
解决方法：  
  
虽然会触发监听，但是不会移动距离。  
  
在要在执行的代码套上if(dy!=0)即可。