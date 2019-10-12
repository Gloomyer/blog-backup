---
title: Android自己留下的深坑-01
date: 2016-8-9 11:02:31
categories: Android
tags:
- Android
- Bug
---
<Excerpt in index | 首页摘要> 
> Android自己留下的深坑  
>
<!-- more -->
<The rest of contents | 余下全文>  
  
简单说，项目中个人中心布局与界面逻辑大体与个人中心是相同的.  
  
为了以后的各方面修改着想，采用的是逻辑判断，复用的是一个fragment.  
  
最近又加上了离线数据缓存(个人中心).  
  
造成了进入用户详情，对面底部有8个RecyclerView tiem布局，回到我的界面会从8个界面闪回(架设个人中心底部有2个)2个RecyclerView item布局。非常生硬.  
  
不够谨慎,自己还以为是Fragment复用导致的，干脆分离开了。结果却是这么坑！~给跪了.  
  
