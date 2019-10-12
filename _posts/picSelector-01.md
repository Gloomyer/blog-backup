---
title: Android 图片选择器
date: 2017-1-5 23:43:42
categories: Android
tags: 
- 图片选择
---
<Excerpt in index | 首页摘要> 
> Android 图片选择器
<!-- more -->
<The rest of contents | 余下全文> 
  
###  项目概述  ###
我们项目中经常碰到这样的需求:选择图片  
  
原生Android提供一套比较完善的机制供开发者调用来选择图片.  
  
但是个各大厂商的深度定制。  
  
我在实际项目中发现，各种系统下返回的方式五花八门。  
  
有直接返回路径的，有返回内容解析者的。甚至还有返回数组，默认给你缩略图的(三星的一款).  
  
再者，系统原生的方式调用UI也是不可定制的。  
  
所以就出现了现在的这个选择器.  
  
项目已经上传至github:[PicSelector](https://github.com/Gloomyer/PicSelector)  
  
让我们看一下Demo图(图片过大，如果无法正常观看可以挂代理或者右键另存到本地查看):  
![图片](http://gloomyer.com/img/img/PicSelector.gif)
  
  
###  使用方式  ###
<font color='red'>需要读取sd卡的权限!</font>  
  
单图选择:  
```java
UIManager.getInstance().start(this, new OnSelectedListener() {
    @Override
    public void onSelect(List<String> selecteds) {
        Toast.makeText(MainActivity.this, selecteds.toString(), Toast.LENGTH_LONG).show();
    }
});
```
  
多图选择:  
```java
//这里的9代表要选择多少张
UIManager.getInstance().start(this, 9, new OnSelectedListener() {
    @Override
    public void onSelect(List<String> selecteds) {
        Toast.makeText(MainActivity.this, selecteds.toString(), Toast.LENGTH_LONG).show();
    }
});
```
  
带记录的模式多图选择:  
```java
//这里的9代表要选择多少张
//history是一个成员变量，类型是List<String> 可以为null(代表没有记录)
UIManager.getInstance().start(this, 9, false, history, new OnSelectedListener() {
    @Override
    public void onSelect(List<String> selecteds) {
        history = selecteds;
        Toast.makeText(MainActivity.this, selecteds.toString(), Toast.LENGTH_LONG).show();
        UIManager.getInstance().removeOnImageClickListener();
    }
});
```
  
额外的一种模式,可以获取图片点击事件.  
如果设置了这个，那么点击图片将执行用户设置的回调,点击预览的右上角才是选择该图片.  
Demo中的第三个按钮，就是这个模式的演示  
请务必在start之前调用!  
```java
//设置
UIManager.getInstance().setOnImageClickListener(new OnImageClickListener() {
    @Override
    public void onClick(String path) {
        Toast.makeText(MainActivity.this, path, Toast.LENGTH_SHORT).show();
    }
});
//取消设置
UIManager.getInstance().removeOnImageClickListener();
```
