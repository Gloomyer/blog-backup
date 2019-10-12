---
title: SwipeRefreshLayout 与 ViewPager事件冲突2
date: 2016-8-5 13:54:00
categories: Android
tags:
- Android
- 自定义View
---
<Excerpt in index | 首页摘要> 
>SwipeRefreshLayout 与 ViewPager事件冲突2  
>
<!-- more -->
<The rest of contents | 余下全文>  
  
[SwipeRefreshLayout 与 ViewPager事件冲突1](http://www.gloomyer.com/2016/07/15/android_swiperefreshlayou_viewpager/)
  
上一篇介绍到解决ViewPager和SwipeRefreshLayout的滑动冲突问题,  
  
通过Vp的PagerChangeAdapter来设置SwipeRefreshLayout的enable来解决.  
  
如果ViewPager是个全屏的，那么没有任何问题，但是如果是轮播图 只占据屏幕1/4左右的高度，并且在顶部的时候，发现滑动仍然不够完美,还是会出现一些问题.  
  
在另一个Android的小哥的提示下，知道了SwipeRefreshLayout有一个属性是负责控制触发它开始执行下拉刷新的距离值.  
  
即当用户上下高度值大于等于这个属性的时候，才会触发SwipeRefreshLayout的滑动事件.  
  
那么就决定在上一次的基础之上，继续加大这个值，来实现完美解决.  
  
这个属性叫做 mTouchSlop 是个int的私有字段.  
  
它的默认值是8，那么通过反射修改其值为100(看自己需求),发现就已经完美解决了这个问题.  
  
看下修改的代码:  
```java
try {
    Class clzz = SwipeRefreshLayout.class;
    Field mTouchSlop = clzz.getDeclaredField("mTouchSlop");
    mTouchSlop.setAccessible(true);
    mTouchSlop.setInt(findSwr, 100);
} catch (Exception e) {
    L.e(TAG, "反射修改值失败.");
}
```