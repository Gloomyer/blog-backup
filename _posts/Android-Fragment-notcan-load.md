---
title: Acitivity重启Fragment不加载问题
date: 2017-02-14 15:33:54
categories: Android
tags:
- Fragment
---
<Excerpt in index | 首页摘要> 
> Acitivity重启Fragment不加载问题
>
> <!-- more -->
> <The rest of contents | 余下全文>  



####  问题描述  #####

项目迭代到现在app体积已经达到了30mb.

一些原来意想不到的问题也接踵而来。

在个别机型（性能较差）的机子上面发现一个问题。

将app挂入后台，去操作的别的应用。

返回之后app所有的地方都是activity根布局的底色！

(后来，反应过来是依赖的Fragment的没有加载)



####  分析问题  ####

大致问题应该是，

当前app挂入后台。

当用户操作其他的app。

发现当前机身内存不够用了！

于是开始杀后台进程。

我们的app也被列入被杀后台。

当用户重启app。

重启了当前的activity。

但是不知为何原因fragment没有被加载！



####  解决  ####

去google之后发现这是fragmentActivity的一个问题。

(我们的base是继承自FragmentActivity)

onSaveInstanceState(Bundle outState);

这个方法，大家应该都不陌生，

是用来保存当前Activity的一些信息，

用于当Activity被重启之后要重复利用的一些数据。



点进FragmentActivity的源码

搜索这个方法，发现FragmentActivity重写了这个方法，并且写了一个备注

```
/**
 * Save all appropriate fragment state.
 */
 保存当前加载的fragment的状态！
```

问题就是这里。

FragmentActivity重写了这个方法，保存了Fragment的状态。

如果当重启Act之后，你的老的Fragment的状态被保存（主要指是Fragment依赖的Act这个 状态被保存了）

你的新Act和Fragment的Act是不一样的。所以导致了不加载。

解决方案就是：

重写onSaveInstanceState(Bundle outState)方法.

然后注释掉super.onSaveInstanceState(outState);//即可