---
title: 利用V7包Style,制作动态效果checkbox
date: 2016-7-7 20:32:10
categories: Android
tags:
- Android
- Style
- V7包
- CheckBox
---
<Excerpt in index | 首页摘要> 
>利用V7包Style,制作动态效果checkbox
>
<!-- more -->
<The rest of contents | 余下全文>  
  
今天模块做到举报功能，另一个Android小哥说他做过了，直接让我跳转即可.  
  
幸福总是来得如此突然...  
  
然后马上接好，跑起来项目，点击去看，我艹，这checkbox的点击效果炸天啊.~  
  
给你们看下效果图:  
![DEMOIMG](http://gloomyer.com/img/img/动态checkbox效果图.gif)
  
马上问Android小哥，怎么做的啊。脑袋完全不够用，根本不知道怎么算的。  
  
Android小哥神秘一笑。偷偷告诉我，这是利用v7包主题来实现的.  
  
赶紧默默打开他的代码开始查看.  
  
看代码理逻辑...  
  
代码对于checkbox多余变化. 在Mainfest文件中发现，含有这个checkbox的Activity注册代码:
```java
	<activity
    android:name=".group.activity.ReportActivity"
    android:screenOrientation="portrait"
    android:theme="@style/AppCompatTheme" />
```

点击AppCompatTheme进去看:  
```java
	<style name="AppCompatTheme" parent="@style/Theme.AppCompat.Light.NoActionBar">  
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="android:windowBackground">@null</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>  
```
重点来了,<font color='red'>colorAccent</font>就是控制点击后checkbox的背景颜色.  
  
马上开了个小demo，照样写了一个，如上图所示，就是这样的界面了!.  
  
注意,@style/Theme.AppCompat.Light.NoActionBar是属于V7包的，所以想要这个效果必须要导入V7包.