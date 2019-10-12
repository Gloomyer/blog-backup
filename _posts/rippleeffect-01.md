---
title: Android触摸反馈动画
date: 2016-11-22 20:49:35
categories: Android
tags: 
- 触摸反馈
- RippleEffect

---
<Excerpt in index | 首页摘要> 
> Android触摸反馈动画
> <!-- more -->
> <The rest of contents | 余下全文> 



##  效果图  ##

![](http://gloomyer.com/img/img/rippleeffect.jpg)

要做底部的这个按钮点击效果。


简单说一下，这个东西是android5.0(API21)之后提出来的概念，叫做触摸反馈。

如果是使用的AppCompat主题并且使用AppCompatActivity，那么一般只要你给按钮添加了点击事件，他就已经封装好了这个。

具体的颜色是在style中修改它

```xml
<item name="colorControlHighlight">@color/colorBackground</item>
```



##  具体实现  ##

先来介绍一下，触摸反馈，一般来说有两种

1：在View本身范围内的扩散动画

2：溢出于View的扩散动画



####  1：在View本身范围内的扩散动画  ####

实现方式1(非Compat主题):

首先，在你的res目录下创建一个drawable-v21的文件夹

里面创建一个ripple.xml

内容如下:

```xml
<?xml version="1.0" encoding="utf-8"?>
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="#fff00000">
    <item>
        <shape android:shape="rectangle">
            <solid android:color="#a0f00000" />
        </shape>
    </item>
    <item>
        <shape android:shape="rectangle">
            <solid android:color="#abcdef" />
        </shape>
    </item>
</ripple>
```

简单说明一下

ripple这个关键字是在v21之后提出的，所以务必在v21以上才使用它!

第二个item:按钮正常状态下的背景

第一个item:按钮备被按住不放的背景

ripple的color:按钮刚开始被触摸，扩展动画的颜色

---

实现方式2(必须Compat主题(最小sdk需为11)):

这个就很简单了，在需要这个这个效果的view上增加如下的属性:

```
android:background="?android:attr/selectableItemBackground"
```

然后在app主题下加入

```xml
<item name="colorControlHighlight">@color/colorBackground</item>
```

这个颜色就是触摸反馈的颜色，缺点就是不能分开控制，必须使用一个颜色（按住不放和涟漪的颜色）



####  2：超出View本身范围内的扩散动画  ####

本身并不难，首先，必须强制继承于AppCompat的主题(至少在v21的版本之上必须!!).

然后你需要创建一个layout-v21的目录

将需要添加点击效果的布局从layout中复制一份到layout-v21目录中去

然后将layout-v21目录下的布局中需要这个效果的View添加如下属性:

```xml
android:background="?android:attr/selectableItemBackgroundBorderless"
```

注意，这个属性必须在v21以上版本才具有，并不想下兼容。包含这个语句的xml在v21以下的版本中出现，会立马崩溃！

具体的颜色也在style中增加这个

```xml
<item name="colorControlHighlight">@color/colorBackground</item>
```



推荐做法:

创建一个values-v21,一个layout-v21目录

在values-v21下创建一个style.xml

然后创建一个与要添加这个效果Activity同名的主题，继承自AppCompat主题。

然后复制要添加点击效果的layout到layout-v21下

给要添加效果的view的xml加入:

```xml
android:background="?android:attr/selectableItemBackgroundBorderless"
```

这样就能保证在v21以上的版本使用，v21一下版本不采用，也不crash!
