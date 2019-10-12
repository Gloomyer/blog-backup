---
title: Android TextView高度问题
date: 2016-9-30 14:49:54
categories: Android
tags: 
- TextView
---
<Excerpt in index | 首页摘要> 
> Android TextView高度问题
<!-- more -->
<The rest of contents | 余下全文> 
  
现在有一个TextView高度为自适应.
他有如此属性:
```xml
<TextView 
    android:id="@+id/item_tv"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="@color/alpha"
    android:text="测试文字"
    android:textColor="@color/middleDark"
    android:textSize="13sp" />
```
我们在写布局的时候，自然希望它的高度也是13dp.  
  
但是往往现实会给我们沉痛的一击.
  
实际上它占用的高度是约为14dp-15dp之间.  
  
为什么?  
  
通过Google得知,TextView有一个属性android:includeFontPadding  
  
它是为了给上标和下标留出足够的空间，默认为真.  
  
原来是预留的padding!.  
  
修改TextViewXml为:  
  
```xml
<TextView 
    android:id="@+id/item_tv"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="@color/alpha"
    android:text="测试文字"
    android:includeFontPadding="false"
    android:textColor="@color/middleDark"
    android:textSize="13sp" />
```
问题解决. 
  
现在产品那边说 当TextView 为两行的时候,觉得距离太过窄了.  
  
通过android:lineSpacingExtra="4dp"我们给TextView设置了行间距
  
意外又来了，TextView的高度又明显增大了.  
  
哪怕TextView显示的文字还是一行，依然会出现问题.  
  
所以一个正常的TextView的高度应该为:  

((textSize + lineSpacingExtra + includeFontPadding) * lineCount);