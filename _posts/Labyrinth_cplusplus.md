---
title: 走迷宫
date: 2017-9-14 16:52:46
categories: Hack
tags:
- Labyrinth
---
<Excerpt in index | 首页摘要> 
> 走迷宫
>
> <!-- more -->
> <The rest of contents | 余下全文>  



####  右手法则，左手法则  #####

最近在看C++,

然后打算利用他做一个走迷宫的小demo。

如图

![](https://gloomyer.com/img/img/labyrinth.gif)


途中T代表小人，空格代表道路，*代表墙壁。

右手法则、左手法则是说，在一个二维的迷宫中。

假设入口出口只有一个的情况下。

从开始你就利用手摸着右（左）边的墙壁，手不要离开墙壁，那么最后你一定能走出去。

大概的意思。

有兴趣的可以去看看代码：

[Github](https://github.com/Gloomyer/Labyrinth_C-)