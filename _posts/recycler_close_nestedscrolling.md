---
title: RecyclerView嵌套导致误触SwipeRefreshLayout 
date: 2016-12-07 15:58:09
categories: Android
tags: 
- RecyclerView
- SwipeRefreshLayout

---
<Excerpt in index | 首页摘要> 
> RecyclerView嵌套导致误触SwipeRefreshLayout
> <!-- more -->
> <The rest of contents | 余下全文> 





经常碰到如下的布局环境

```xml
<SwipeRefreshLayout>
	<RecyclerView />
</SwipeRefreshLayout>
```

用来实现下拉刷新.

但是当内部的这个RecyclerView内容复杂，涉及到RecyclerView嵌套RecyclerView的时候。

你就会发现内部的RecyclerView也会触发下拉刷新。（应该是嵌套滑动导致的）



应该说这是RecyclerView的Bug.

过去的解决方案是:

在外部RecyclerView的onScrollListener中加入这句话

```java
mRefreshLayout.setEnabled(mLayoutManager
                            .findFirstCompletelyVisibleItemPosition() == 0);
```

但是我发现在一些情况下依然能够触发，而且 个别机型上findFirstCompletelyVisibleItemPosition会返回-1.

导致mRefreshLayout不能被触发了.

---

<font color='red'>最后感觉最完美的解决方案如下:</font>

内部被嵌套的RecyclerView(一般用于九宫图显示)自定义继承自RecyclerView

然后在初始化设置

```java
setNestedScrollingEnabled(false); //关闭嵌套滚动
GridLayoutManager gridLayoutManager = new GridLayoutManager(
                context, 3, GridLayoutManager.VERTICAL, false) {
  @Override
  public boolean canScrollVertically() {
    return false;
  }

  @Override
  public boolean canScrollHorizontally() {
    return false;
  }
};
```

