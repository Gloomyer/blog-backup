---
title: Android部分国产机型RecyclerView自动notify问题
date: 2018-03-06 20:43:21
categories: Android
tags: 
- recyclerview
---
<Excerpt in index | 首页摘要> 
> ndroid部分国产机型RecyclerView自动notify问题
> <!-- more -->
> <The rest of contents | 余下全文> 

##  写在前面 ##
最近app启动偶先crash情况，尤其小米机型最为严重。

经过log发现崩溃在recyclerview bindViewHolder身上，

bindViewHolder中加载图片Glide的target为null导致的

但是imageview是从itemview身上findviewById获取的，理论上是不可能为空的。

##  问题所在 ##
首先出现这个情况你必须这个adapter中包含多种类型的type

bug复现操作:
```
mDatas.add(data(3个type1的数据)) (mDatas 是adapter绑定的数据源)
//一些操作 但是没有进行notifyitemchange操作
mDatas.add(data(3个type2的数据、或者包含一个type2的数据))
adapter.notifyItemChange()
```

大致问题是这样的，在第二次向mDatas中添加数据的时候，Recyclerview自动调用了RecyclerView （注意，但是第二次数据没有添加进去！） 调用了notify之后再加入的数据，我们又执行了notify操作。

这样在type1创建出来的viewholder身上执行了bind type2的viewHolder

这个bug很难说是谁的过，因为既然自动帮我们调用了 那么recyclerview也不应该出现这个情况，但是总的来说如果recyclerview不自动调用应该也不会出现这个问题。

通过断点进入recyclerview 发现进去的点都在注释身上，说明应该是小米的rom对recyclerview进行了一些修改。所以这个锅也可以说小米rom应该背

##  修复方式 ##
创建临时缓存 两次的输入先加入tempDatas， 加入完成了再统一加入mDatas中去。

最后我们自己执行notifyItemChange() 避免让recyclerview帮我们调用。