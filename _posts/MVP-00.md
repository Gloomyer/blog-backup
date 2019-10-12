---
title: 浅谈MVP
date: 2016-10-13 11:13:44
categories: Android
tags: 
- MVP
---
<Excerpt in index | 首页摘要> 
> 浅谈MVP
> <!-- more -->
> <The rest of contents | 余下全文> 

##  引言  ##

**Model-view-presenter**，简称[MVP](https://zh.wikipedia.org/wiki/MVP)，是电脑软件设计工程中一种对针对[MVC](https://zh.wikipedia.org/wiki/Model-view-controller)模式，再审议后所延伸提出的一种软件设计模式。

####  模式描述  ####

Model-view-presenter (MVP)是使用者界面设计模式的一种，被广范用于便捷自动化单元测试和在呈现逻辑中改良分离关注点（separation of concerns）。

- **Model** 定义使用者界面所需要被显示的资料模型，一个模型包含着相关的业务逻辑。
- **View** 视图为呈现使用者界面的终端，用以表现来自 Model 的资料，和使用者命令路由再经过 Presenter 对事件处理后的资料。
- **Presenter** 包含着元件的事件处理，负责检索 Model 取得资料，和将取得的资料经过格式转换与 View 进行沟通

[来自维基百科](https://zh.wikipedia.org/wiki/Model_View_Presenter)

上面是摘自维基百科对于MVP的概述。

谈谈我个人的理解。

在Android开发中，我们通常使用的是MVC设计模式(过去).

Android项目结构也是很好的MVC架构,Layout布局负责VIew的展示，Activity/Fragment做为控制,JavaBean负责Model.

但是随着现在的项目逻辑复杂度的提升，往往在Activity中又负责View的显示，又负责Control，又负责View。在新功能的开发周期中，往往这样做的速度是非常快的，但是在后期功能的改进以及迭代中 我们要从上千行的Activity中去寻找某一个点的功能修改、增加是相当痛苦的。

所以慢慢的衍生出了现在的MVP。



##  好处  ##

既然说到因为逻辑复杂，后期迭代困难等原因衍生出来的MVP设计模式。

那么他的好处到底在哪里呢？

####  逻辑清晰  ####

是的，MVP中将传统的Activity/Fragment和layout布局只用于View的展示，将所有的逻辑处理控制代码通通迁移至P层(presenter,后面将统称P层).

V层(View) <-> P层 <-> M层(Model)

将控制、逻辑处理、网络请求、数据绑定等一概操作通通交予P层。

Acitivity是负责UI处理。

互相之间调用通过接口回调的方式来实现。

将过去近千行代码的Activity优化为200-300行。

PS:

​	P层根据相应的方法去做相应的事情,感觉会有一些函数式编程的意思在里面.



##  坏处  ## 

只谈好处不谈坏处都是耍流氓！

为了将Activity合理的抽出，那么不可避免的分包结构上就要复杂许多，这是一点。

各层直接通过回调的调用，那么最好申明接口来实现。这就增加了一定量的代码量！

而开发中，往往在最初设计的时候容易遗漏掉其中某一个功能点，做到那个功能点的时候又想起来了，这个时候你需要去增加接口方法，回到实现实现接口方法。又提升了新功能开发的难度。



##  总结  ##

简单概括一下，

MVP是对于迭代开发相当友好，

尔相应的对于新功能开发相当不友好的。

感觉外包公司估计会较少的选择MVP。

总的来说，MVP是一种相当实用的设计模式，对于在涉及到多次迭代开发的项目是相当值得采用的！



个人理解，如果有不当的地方。敬请指出，不胜感激!