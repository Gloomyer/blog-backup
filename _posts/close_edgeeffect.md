---
title: 关闭滚动View滚动到边界的动画
date: 2016-11-17 15:36:10
categories: Android
tags: 
- EdgeEffect
- ScrollView

---
<Excerpt in index | 首页摘要> 
>关闭ListView/ScrollView/RecyclerView/GridView
>
>滚动到边界的阴影效果
><!-- more -->
><The rest of contents | 余下全文> 

要做的一个界面:
![](http://gloomyer.com/img/img/EdgeEffect.jpg)



底部和顶部如果继续拉动会出现上图的这个阴影效果，这个阴影颜色是根据主题颜色来走的。

如果在上面的这种界面中出现会非常的难看。

如何关闭呢？

也很简单 在ListView/RecylerView 类似可以滚动的View xml中加入如下的属性

```xml
android:fadingEdge="none" //API 13之前
android:overScrollMode="never" //API 13 之后
```

第一条在api13之后就废弃了google推荐使用第二句。

谨慎起见，两句都配置吧！



参考资料[stackoverflow](http://stackoverflow.com/questions/4130461/is-there-a-way-to-disable-edit-the-fading-that-a-list-view-has-at-its-edges)