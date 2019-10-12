---
title: SwipeRefreshLayout 与 ViewPager事件冲突
date: 2016-7-15 15:13:21
categories: Android
tags:
- Android
- 自定义View
---
<Excerpt in index | 首页摘要> 
>SwipeRefreshLayout 与 ViewPager事件冲突  
>
<!-- more -->
<The rest of contents | 余下全文>  
  
项目下拉刷新使用的是SwipeRefreshLayout  
  
刚好有一个页面顶部是ViewPager实现的轮播图  
  
发现SwipeRefreshLayout会将ViewPager的事件抢走，导致轮播图手动滑动非常困难。  
  
先是尝试通过分析上下滑动和左右滑动来指定是否接受事件，可是效果并不是很好  
  
网上搜索发现一个取巧的办法。  
  
ViewPager 有个监听方法的回调，是onPagerChangeListener  
  
这是ViewPager页面切换的时候会触发的一个回调，里面有一个方法，是onPageScrollStateChanged(int state)  
  
这个方法是ViewPager滚动状态改变触发的方法，state=1：正在滚动，0：没有变化，2：滑动完毕  
  
那么通过监听这个回调，来设置SwipeRefreshLayout的Enable，就可以了。  
  
```java  
vp.setOnPageChangeListener(new ViewPager.OnPageChangeListener() {
    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {

    }

    @Override
    public void onPageSelected(int position) {

    }

    @Override
    public void onPageScrollStateChanged(int state) {
        if (state == 1) {
            swr.setEnabled(false);
        } else {
            swr.setEnabled(true);

        }
    }
});
```