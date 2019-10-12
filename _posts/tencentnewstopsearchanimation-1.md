---
title: 模仿腾讯新闻首页顶部搜索框的拖拽效果
date: 2016-8-25 17:59:16
categories: Android
tags:
- Android
- 自定义View
---
<Excerpt in index | 首页摘要> 
> 模仿腾讯新闻首页顶部搜索框的拖拽效果
>
<!-- more -->
<The rest of contents | 余下全文>  
  
项目打算模仿腾讯新闻首页顶部搜索的效果.先看下效果图吧.  
  
![](http://www.gloomyer.com/img/img/tencentnewstopanimation.gif)  
(底部有demo下载地址)
  
说一下实现思路.  
  
布局:  
```
	<帧布局>
		<顶部View />
		
		<下拉刷新组件>
			<RecyclerView />
		</下拉刷新组件>
	</帧布局>  
```
帧布局采用我们自己写的一个MFrameLayout  
  
然后重写  

截断事件 与 触摸事件 方法  
  
在截断事件中判断当前一些上下文，来判断是否截断事件.  
  
然后在触摸事件方法中去控制 设置 底部的View移动，当达到某些条件.  
  
将下拉事件传递给下载刷新组件.  
  
具体实现请移步[Demo](https://github.com/Gloomyer/DragTopDemo)