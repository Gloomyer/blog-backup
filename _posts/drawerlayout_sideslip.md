---
title: Drawerlayout实现侧拉菜单
date: 2017-04-17 10:25:03
categories: Android
tags:
- Drawerlayout
---
<Excerpt in index | 首页摘要> 
> 利用Drawerlayout实现侧拉菜单
>
> 超级简单
>
> <!-- more -->
> <The rest of contents | 余下全文>  

#### 概述

过去我们实现侧拉操作有一个库帮助我们实现了很好的封装。

它就是[SlideMenu](https://github.com/TangKe/SlideMenu)

再后来，谷歌发现这类侧拉操作很多，

就在支持包中提供了DrawerLayout(侧拉)

但是如果我们UI要求的是侧滑，DrawerLayout原生是不支持的。

侧拉(类似原生的DrawerLayout)

侧滑(类似QQ侧滑菜单)

但是DrawerLayout提供了很好的扩展支持，我们可以非常容易的实现侧拉！

#### 效果图

让我们先来看看效果图:

![](http://gloomyer.com/img/img/drawerlayout_sideslip.gif)

#### UI代码

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <android.support.v7.widget.Toolbar
        android:id="@+id/mToolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="@color/colorPrimary" />

    <android.support.v4.widget.DrawerLayout
        android:id="@+id/mDrawerLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.v7.widget.RecyclerView
            android:id="@+id/mRecyclerView"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />


        <FrameLayout
            android:layout_width="200dp"
            android:layout_height="match_parent"
            android:layout_gravity="start"
            android:background="#abcdef">

            <TextView
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:gravity="center"
                android:text="测试菜单" />

        </FrameLayout>

    </android.support.v4.widget.DrawerLayout>


</LinearLayout>

```



MainActivity核心代码:

```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Toolbar mToolbar = (Toolbar) findViewById(R.id.mToolbar);
        setSupportActionBar(mToolbar);

        DrawerLayout mDrawerLayout = (DrawerLayout) findViewById(R.id.mDrawerLayout);
		//这个是ActionBar左上角的动画按钮
        ActionBarDrawerToggle toggle = new ActionBarDrawerToggle(this, mDrawerLayout, mToolbar,R.string.open,R.string.close);
        toggle.syncState();
        mDrawerLayout.addDrawerListener(toggle);
      	//这里是实现的具体逻辑
      	//这个DrawerListener是DrawerLayout的一些事件产生之后，DrawerLayout给我们的回调
        mDrawerLayout.addDrawerListener(new MyDrawerListener());




		//主界面的数据绑定操作 代码就不贴了
        RecyclerView mRecyclerView = (RecyclerView) findViewById(R.id.mRecyclerView);
        mRecyclerView.setLayoutManager(new LinearLayoutManager(this));
        initData();
        mRecyclerView.setAdapter(new MyAdapter());
    }
```



#### 核心代码

```java
package com.gloomyer.minademo;

import android.support.v4.widget.DrawerLayout;
import android.view.View;

/**
 * 恻拉变侧滑
 */

class MyDrawerListener implements DrawerLayout.DrawerListener {
    private static final String TAG = "MyDrawerListener";

    private View contentView = null;
    private float moveSize;

  	//这个是在DrawerLayout在拉动菜单的时候产生的回调操作
  	//主要是第二个参数，它代表了当前的菜单滑动值，取值为0-1
    @Override
    public void onDrawerSlide(View drawerView, float slideOffset) {
        if (contentView == null) {
            DrawerLayout mDrawerLayout = (DrawerLayout) drawerView.getParent();
            //获取正常的contentView
            contentView = mDrawerLayout.getChildAt(0);

            //计算contentView和恻拉菜单的占比关系
          	//即当侧拉菜单完全拉出，contentView需要移动的具体距离
            moveSize = mDrawerLayout.getChildAt(1).getWidth();
        }

        //利用属性值移动contentView
        contentView.setTranslationX(slideOffset * moveSize);
    }

    @Override
    public void onDrawerOpened(View drawerView) {

    }

    @Override
    public void onDrawerClosed(View drawerView) {

    }

    @Override
    public void onDrawerStateChanged(int newState) {

    }
}

```

