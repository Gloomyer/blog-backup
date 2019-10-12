---
title: SwipeRefreshLayout自动刷新
date: 2016-6-24 17:33:26
categories: Android
tags:
- Android
- SwipeRefreshLayout
---
<Excerpt in index | 首页摘要> 
>SwipeRefreshLayout进入自动刷新

<!-- more -->
<The rest of contents | 余下全文>  
需求：当前页面一进入就需SwipeRefreshLayout为正在刷新状态  
  
SwipeRefreshLayout.setRefreshing(true);无效  
  
知乎上找到解决方案：  
```java
	SwipeRefreshLayout.post(new Runnable(){
		SwipeRefreshLayout.setRefreshing(true);
	});  
```
这样才生效，但是请注意，这不会去执行refreshing的listener  
  
如果需要执行监听需要自己手动调用一次listener。  
  
PS，目前还没有查看源码具体原理还不是特别清楚。  
估计是在view未填充完毕，需要什么参数未初始化导致的。