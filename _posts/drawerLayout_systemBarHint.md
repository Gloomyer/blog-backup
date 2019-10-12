---
title: DrawerLayou导致无法设置状态栏透明
date: 2016-11-11 15:49:39
categories: Android
tags: 
- DrawerLayou
- SystemBarTintManager
---
<Excerpt in index | 首页摘要> 
>DrawerLayou导致无法设置状态栏透明
>无法将app标题顶至状态栏
><!-- more -->
><The rest of contents | 余下全文> 

要实现的效果图:
![](http://gloomyer.com/img/img/DrawerLayout_SystemBarTintManager.jpg)



设置状态栏背景色alpha为0，将app标题栏顶部顶到状态栏，是用的SystemBarTintManager库实现。

现在这个节目要增加一个侧拉菜单，打算用DrawerLayout来实现。

然后发现app标题栏顶不到状态栏了。

设置fitsSystemWindows也不行.

百度后知道DrawerLayout是不能设置fitsSystemWindows属性的。



解决方案:

```
root布局不要使用DrawerLayout
在DrawerLayout外层套一个FrameLayout即可
```

