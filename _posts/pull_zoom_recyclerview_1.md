---
title: RecyclerView实现下拉放大
date: 2017-04-24 17:12:29
categories: Android
tags:
- RecyclerView
- PullZoomView
---
<Excerpt in index | 首页摘要> 
> RecyclerView实现下拉放大
>
> <!-- more -->
> <The rest of contents | 余下全文>  

#### 概述

经常有这样的产品需求。

某个界面顶部16 ：9的大图

要求在顶部向下拽的时候缩放这个大图

来实现下拉放大的效果

#### 效果图

![demo图](http://gloomyer.com/img/img/pull_zoom_recycler_view_demo.gif)

#### 思路

首先封装adapter给RecyclerView添加一个头部

然后封装一个我们自己的RecyclerView

重写onTouch()方法

这里重点说下。

最开始我的想法是利用属性动画来放大。

时候发现是可以放大。但是会造成header和item1 重叠。

所以这里横向放大用的属性动画

纵向放大是直接修改的header的height来实现的。



#### 具体代码

先看Act:

```java
package com.gloomyer.pullzoomdemo;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {

    private MyRecyclerView mRecyclerView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mRecyclerView = (MyRecyclerView) findViewById(R.id.mRecyclerView);
        mRecyclerView.setAdapter(new MyAdapter());
    }

    public int getWidht() {
        try {
            return getWindowManager().getDefaultDisplay().getWidth();
        } catch (Exception e) {
        }
        return 720;
    }

    class MyHolder extends RecyclerView.ViewHolder {
        TextView tv;

        public MyHolder(View itemView) {
            super(itemView);
            if (itemView instanceof TextView)
                tv = (TextView) itemView;

        }
    }

    private class MyAdapter extends RecyclerView.Adapter<MyHolder> {
        @Override
        public MyHolder onCreateViewHolder(ViewGroup parent, int viewType) {
            View view = LayoutInflater.from(parent.getContext())
                    .inflate(
                            viewType == 0
                                    ? android.R.layout.simple_list_item_1
                                    : R.layout.header, parent, false);
            if (viewType == 1) {
                view.getLayoutParams().height = (int) (getWidht() / 16.0f * 9);//修改header为16:9
                mRecyclerView.setZoomView(view);
            }
            return new MyHolder(view);
        }

        @Override
        public void onBindViewHolder(MyHolder holder, int position) {
            if (getItemViewType(position) == 0) {
                holder.tv.setText("pos:" + position);
            }
        }

        @Override
        public int getItemViewType(int position) {
            if (position == 0)
                return 1;
            return 0;
        }

        @Override
        public int getItemCount() {
            return 30;
        }
    }
}

```

非常简单。

header就是一个imageview

```xml
<?xml version="1.0" encoding="utf-8"?>
<ImageView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="180dp"
    android:background="#fff"
    android:scaleType="fitXY"
    android:src="@drawable/pic" />
```



重点在我们封装的RecyclerView

```java
package com.gloomyer.pullzoomdemo;

import android.content.Context;
import android.support.annotation.Nullable;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.util.AttributeSet;
import android.util.Log;
import android.view.MotionEvent;
import android.view.View;
import android.view.ViewConfiguration;

/**
 * Created by Gloomy on 2017/4/24.
 */

public class MyRecyclerView extends RecyclerView {

    private static final String TAG = "MyRecyclerView";
    private LinearLayoutManager mLayoutManager;
    private int mTouchSlop;
    private View zoomView;

    public MyRecyclerView(Context context) {
        this(context, null);
    }

    public MyRecyclerView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public MyRecyclerView(Context context, @Nullable AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        mLayoutManager = new LinearLayoutManager(context);
        setLayoutManager(mLayoutManager);
        mTouchSlop = ViewConfiguration.get(context).getScaledTouchSlop();
    }

    public void setZoomView(View v) {
        this.zoomView = v;
    }

    private float downY;

    int countSize;

    int defaultHeight;

    @Override
    public boolean onTouchEvent(MotionEvent e) {
        if (countSize == 0
                && zoomView != null) {
            defaultHeight = zoomView.getHeight();//记录zoomView的原本高度
            countSize = getHeight() - zoomView.getHeight();//计算缩放高度计算对比值
            zoomView.setPivotX(zoomView.getWidth() / 2);//缩放点
            zoomView.setPivotY(0);//缩放点
            zoomView.invalidate();//缩放点生效
            Log.e(TAG, "height:" + countSize);
        }
        if (mLayoutManager.findFirstCompletelyVisibleItemPosition() == 0
                && zoomView != null) {
            if (e.getAction() == MotionEvent.ACTION_DOWN) {
                downY = e.getY();
            } else if (e.getAction() == MotionEvent.ACTION_MOVE) {
                //移动
                float moveY = e.getY();
                int dY = (int) (moveY - downY);
                boolean isScroll = Math.abs(dY) > mTouchSlop;

                if (isScroll) {
                    if (dY > 0) {
                        //缩放
                        Log.i(TAG, "xxxx:" + (dY - mTouchSlop));
                      	//缩放值计算方式，可以根据自己的方式来算。
                        float scroll = (dY - mTouchSlop) * 0.8f / countSize;
                        scroll *= 1.0;
                        scroll += 1;
                        //Log.e(TAG, "scroll: " + scroll);
                        zoomView.setScaleX(scroll);
                      	//高度缩放直接修改高度值
                        zoomView.getLayoutParams().height = (int) (defaultHeight * scroll);
                      //这里需要通知刷新，否则会出问题
                        if (getAdapter() != null)
                            getAdapter().notifyDataSetChanged();
                        return true;
                    }
                }
            } else if (e.getAction() == MotionEvent.ACTION_UP
                    || e.getAction() == MotionEvent.ACTION_CANCEL) {
                //回复
                zoomView.setScaleY(1);
                zoomView.getLayoutParams().height = defaultHeight;
                if (getAdapter() != null)
                    getAdapter().notifyDataSetChanged();
                invalidate();
            }
        }
        return super.onTouchEvent(e);
    }


}

```

