---
title: 滚动View内部嵌套RecyclerView
date: 2016-12-27 17:19:29
categories: Android
tags:
- RecyclerView
---
<Excerpt in index | 首页摘要> 
> 滚动View内部嵌套RecyclerView  
>
> <!-- more -->
> <The rest of contents | 余下全文>  

之前写过一篇RecyclerView嵌套RecyclerView的[博客](http://gloomyer.com/2016/06/23/android-recycleviewitemheight-01/)

之前用的方式是代码计算并控制内部RecyclerView的高度来实现的。



更加完美的解决方案为:


LayoutManager身上有如下一个方法(v23.2.0以及以上版本才有)

```
setAutoMeasureEnabled(true);
//如果这个设置了true，并且RecyclerView高度为wrap_content
//那么RecyclerView的高度将会由Item的高度来决定!
```

然后再将内部的RecyclerView的嵌套滑动关闭(避免消费不必要的事件)

```java
setNestedScrollingEnabled(false);
```

这样才是滚动View嵌套RecyclerView的完美解决方案.