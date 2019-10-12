---
title: 不知道模仿哪里的，滑动取消.
date: 2017-5-15 12:41:00
categories: Android
tags: 
- 滑动取消
---
<Excerpt in index | 首页摘要> 
> 不知道设计从哪里开到的，模仿哪里的，滑动取消.
> <!-- more -->
> <The rest of contents | 余下全文> 

####  引言  ####
最近因为一些事情，断更了一段时间。

刚好，朋友上周找我，他们设计不知道从哪里看了个效果。

他搞不定找我帮忙。

索性闲来无事，就做了一个demo.

#### 设计图稿 ####
~![](http://gloomyer.com/img/img/swipe-cancel-01.gif)

#### Demo图稿 ####
先看一下demo，当然比较难看，配色也很丑。
功能为主，界面为辅。不要太在意。
~![](http://gloomyer.com/img/img/swipe-cancel-02.gif)

#### 技术实现难点 ####
底部的实现是利用属性动画的scaleX来完成的
那么就出现了一个问题。
当缩放底部的按钮的时候，里面的文字也被一起缩放了。

解决方案也不是很难。
<帧布局>
 <文字>
</帧布局>
实际上的布局是这样的。
然后当缩放帧布局由大变小的时候,
我们就放大TextView
根据帧布局的缩放比例放大TextView

#### 代码实现  ####
先看一下mainAct的布局
```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!" />

    <com.gloomyer.swipedemo.MyGroup
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:layout_gravity="bottom">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:layout_marginRight="150dp"
            android:background="#f0f0f0"
            android:gravity="center"
            android:text="接单中"
            android:textSize="16sp" />

        <FrameLayout
            android:layout_width="match_parent"
            android:layout_height="50dp"
            android:layout_gravity="right"
            android:background="#f0f">

            <TextView
                android:id="@+id/textView"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:gravity="center"
                android:text="开始接单"
                android:textSize="16sp" />
        </FrameLayout>

    </com.gloomyer.swipedemo.MyGroup>
</FrameLayout>

```
MainAct代码就是加载布局，没有其他的了就不贴上来了
重点是这个MyGroup的代码
```java
package com.gloomyer.swipedemo;

import android.animation.Animator;
import android.animation.ObjectAnimator;
import android.content.Context;
import android.graphics.Rect;
import android.os.Handler;
import android.support.annotation.AttrRes;
import android.support.annotation.NonNull;
import android.support.annotation.Nullable;
import android.util.AttributeSet;
import android.util.Log;
import android.view.MotionEvent;
import android.view.View;
import android.view.ViewConfiguration;
import android.view.ViewGroup;
import android.view.WindowManager;
import android.widget.FrameLayout;
import android.widget.TextView;

/**
 * Created by Gloomy on 5/13/2017.
 */

public class MyGroup extends FrameLayout implements View.OnClickListener {

    private static final String TAG = "MyGroup";
    public static final int ANIMATOR_TIME = 300;
    private View swipeView;
    private Rect swipeRect;
    private int mTouchSlop;
    private int openWidth;
    private int closeWidth;
    private boolean status;//true:未接单.false：接单中
    private Handler mHandler;
    private boolean is2;


    public MyGroup(@NonNull Context context) {
        this(context, null);
    }

    public MyGroup(@NonNull Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public MyGroup(@NonNull Context context, @Nullable AttributeSet attrs, @AttrRes int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mTouchSlop = ViewConfiguration.get(context).getScaledTouchSlop();
        mHandler = new Handler();
        status = true;
    }

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();

        if (swipeView == null)
            swipeView = getChildAt(1);

        swipeView.setOnClickListener(this);
        WindowManager wm = (WindowManager) getContext()
                .getSystemService(Context.WINDOW_SERVICE);
        closeWidth = wm.getDefaultDisplay().getWidth();
        openWidth = (int) (closeWidth * 1.0f / 3);


        //设置容器里面的View宽度比例为:2:1
        ViewGroup.LayoutParams layoutParams = getChildAt(0).getLayoutParams();
        layoutParams.width = (int) (closeWidth * 1.0f / 3 * 2);
        getChildAt(0).setLayoutParams(layoutParams);
        layoutParams = swipeView.getLayoutParams();
        layoutParams.width = closeWidth;
        swipeView.setLayoutParams(layoutParams);

        //((TextView)((ViewGroup)swipeView).getChildAt(0)).setText("开始接单");
        //默认开启开始接单功能
        post(new Runnable() {
            @Override
            public void run() {
                swipeView.setPivotX(swipeView.getRight());
                swipeView.setPivotY(swipeView.getBottom() / 2);
                swipeView.invalidate();
            }
        });
    }

    float downX;
    float nowWidth;

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        if (status)
            return super.onTouchEvent(event);

        Log.i(TAG, "action: " + event.getAction());
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN: {
                downX = event.getX();
                is2 = swipeRect.contains((int) event.getX(), (int) event.getY());
                nowWidth = getWidth() - swipeRect.left;
                Log.i(TAG, "x,y:" + (int) event.getX() + " , " + (int) event.getY() + "   isTrue:" + is2);
                break;
            }
            case MotionEvent.ACTION_MOVE: {
                int moveX = (int) ((event.getX() - downX) * (getWidth() * 1.0f / (closeWidth - openWidth)));
                if (Math.abs(moveX) > mTouchSlop) {
                    //构成滑动
                    float w = this.nowWidth - moveX;

                    if (w >= closeWidth)
                        w = closeWidth;
                    if (w <= openWidth)
                        w = openWidth;

                    swipeRect.left = (int) (swipeRect.right - w);
                    float scale = w / closeWidth;
                    //Log.i(TAG, "滑动:" + moveX + " , tWidth:" + this.nowWidth + " , w:" + w);
                    swipeView.setScaleX(scale);

                    float scaleX = swipeView.getScaleX();
                    Log.i(TAG, "scaleX:" + scaleX);

                    if (scale == 1.0f)
                        scaleX = 1.0f;
                    else
                        scaleX = 1.0f / scaleX;

                    View v = ((ViewGroup) swipeView).getChildAt(0);
                    v.setScaleX(scaleX);
                }
                break;
            }
            case MotionEvent.ACTION_UP: {
                Log.i(TAG, "up");
                if (swipeRect.left != openWidth
                        && swipeView.getLeft() != closeWidth) {
                    //获取一个判断值，用于判断是否取消,其值为整体2/3 的一半
                    int value = (int) (getWidth() * 1.0f / 3 * 2 / 2);
                    Log.i(TAG, "ss" + (swipeRect.left > value));
                    if (swipeRect.left > value) {
                        //不达成取消
                        setStatus(false);
                    } else {
                        //达成取消
                        setStatus(true);
                    }
                }
                break;
            }
        }


        return true;
    }


    @Override
    public boolean onInterceptTouchEvent(MotionEvent event) {
        if (status)
            return super.onInterceptHoverEvent(event);
        return true;
    }

    @Override
    public void onClick(View v) {
        if (swipeView != null && v == swipeView) {
            if (status) {
                //进入接单状态
                Log.i(TAG, "进入接单状态");
                setStatus(false);
            }

            return;
        }
    }

    private void setStatus(boolean status) {
        this.status = status;

        TextView tv = (TextView) ((ViewGroup) swipeView).getChildAt(0);

        if (!status) {
            //进入接单状态
            float scale = openWidth * 1.0f / closeWidth;
            ObjectAnimator animator = ObjectAnimator.ofFloat(swipeView, "scaleX", swipeView.getScaleX(), scale);
            ObjectAnimator animator2 = ObjectAnimator.ofFloat(tv, "scaleX", tv.getScaleX(), 1.0f / scale);

            animator.setDuration(ANIMATOR_TIME);
            animator2.setDuration(ANIMATOR_TIME);
            animator.start();
            animator2.start();
            tv.setText("取消接单");
            swipeRect = new Rect((int) (getWidth() * 1.0f / 3 * 2), 0, getWidth(), getHeight());
        } else {
            ObjectAnimator animator = ObjectAnimator.ofFloat(swipeView, "scaleX", swipeView.getScaleX(), 1.0f);
            ObjectAnimator animator2 = ObjectAnimator.ofFloat(tv, "scaleX", tv.getScaleX(), 1.0f);
            animator.setDuration(ANIMATOR_TIME);
            animator2.setDuration(ANIMATOR_TIME);
            animator.start();
            animator2.start();
            tv.setText("开始接单");
            swipeRect = new Rect(0, 0, getWidth(), getHeight());
        }


    }
}

```