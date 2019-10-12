---
title: RecyclerView小技巧-1
date: 2016-8-2 11:18:37
categories: Android
tags:
- Android
- RecyclerView
- GridLayoutManager
---
<Excerpt in index | 首页摘要> 
> RecyclerView设置实现自动加载  
> RecyclerView设置GridLayoutManager实现可控制itemSpan
<!-- more -->
<The rest of contents | 余下全文>  
  
项目对于对于数据的集合展示抛弃了ListView，使用了更加优秀的RecycleView来实现.  
  
项目某些模块属于"动态浏览"界面,需要进行分页加载，但是又希望用户阅读更多的动态,所以经过考虑，要求我们实现自动加载(既用户浏览到倒数第一条动态就开始自动去加载下一页数据)  
  
页面大概长这样:  
  
![GUI](http://gloomyer.com/img/img/RecyclerView-skill01.png)  
  
RecyclerView设置是GridLayoutManager,其itemSpan是2
  
设置让加载中View独占一行的方式为：  
```java
	GridLayoutManager layoutManager =
            new GridLayoutManager(MyApp.getInstance()
                    , 2, GridLayoutManager.VERTICAL, false);
    layoutManager.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
        @Override
        public int getSpanSize(int position) {
			//重点在这里，position代表item的坐标，返回2就代表占用2个span
            if (position == starPlans.size())
                return 2;
            return 1;
        }
    });  
```
布局实现了，然后就是自动加载，一想到滑动到最后一个条目，就应该知道了肯定是利用RecycleView的onScrollListener.  
  
我是这么做的:
```java
	recyclerView.addOnScrollListener(new RecyclerView.OnScrollListener() {
        @Override
        public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
            super.onScrolled(recyclerView, dx, dy);
            if ((getLastVisibleViewPosition() >= starPlans.size() - 1)
                    && !isLoadMore
                    && footerView != null
                    && footerStatus) {
                getCategoryData();
            }
        }

		/**
		 * 获取当前RecycleView最后一个可见的item坐标
		/*
		private int getLastVisibleViewPosition() {
	        LinearLayoutManager layoutManager = (LinearLayoutManager) recyclerView.getLayoutManager();
	        return layoutManager.findLastVisibleItemPosition();
   		}
    });  
```
当同时满足:最后一个可是坐标是属于当前item的最后一个，没有正在执行加载任务，footerView已经被创建出来，footerView的状态是可以加载下一页的状态，就执行加载下一页任务.
  
以上。完毕.