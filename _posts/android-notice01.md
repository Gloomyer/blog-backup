---
title: Android上下滚动通知自定义View
date: 2016-6-24 17:33:26
categories: Android
tags:
- Android
- 自定义View
---
<Excerpt in index | 首页摘要> 
>Android自定义View实现上下滚动通知
<!-- more -->
<The rest of contents | 余下全文>  
朋友项目需要个上下滚动通知的通知View，没有什么思路，求助于我。  
  
简单说下我的思路：  
  
自定义View继承至FrameLayout  
  
内部包含要滚动的View，默认Visibility为invisible。  
  
然后利用平移动画来移动View，在动画开始/结束的时候设置view的可视，在某一个view消失动画执行的时候开始启动下一个view的进入动画。  
  
废话不多说，直接看代码。  
  
要滚动View的XML：  
```xml
	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="wrap_content"
	    android:orientation="vertical">
	
	    <TextView
	        android:id="@+id/tv_title"
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:gravity="center"
	        android:text="标题"
	        android:textSize="16sp" />
	
	    <TextView
	        android:id="@+id/tv_value"
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:layout_marginTop="5dp"
	        android:gravity="center"
	        android:text="公告内容"
	        android:textSize="12sp" />
	</LinearLayout>  
```
自定义的代码：  
```java
	package com.gloomyer.toticedemo;
	
	import android.content.Context;
	import android.os.Handler;
	import android.util.AttributeSet;
	import android.view.View;
	import android.view.animation.Animation;
	import android.view.animation.TranslateAnimation;
	import android.widget.FrameLayout;
	import android.widget.TextView;
	
	import java.util.List;
	
	/**
	 * 滚动通知栏
	 */
	public class NoticeVIew extends FrameLayout {
	    View view1;
	    View view2;
	    private List<Bean> list;
	    private int index = 0;
	
	    public NoticeVIew(Context context, AttributeSet attrs) {
	        super(context, attrs);
	        view1 = View.inflate(context, R.layout.item, null);
	        view2 = View.inflate(context, R.layout.item, null);
	
	        view1.setVisibility(View.VISIBLE);
	        view2.setVisibility(View.INVISIBLE);
	        addView(view1);
	        addView(view2);
	    }
	
	    public void setData(List<Bean> list) {
	        this.list = list;
	        index = 0;
	        start1();
	    }
	
	    private static Handler mHandler = new Handler();
	
	    private void setData(View view) {
	        TextView title = (TextView) view.findViewById(R.id.tv_title);
	        TextView value = (TextView) view.findViewById(R.id.tv_value);
	        title.setText(list.get(index).title);
	        value.setText(list.get(index).value);
	        index++;
	        if (index == list.size())
	            index = 0;
	    }
	
	    private void start1() {
	        final View view = view1;
	
	        Animation anim = getAnim(true);
	        anim.setAnimationListener(new Animation.AnimationListener() {
	            @Override
	            public void onAnimationStart(Animation animation) {
	                view.setVisibility(View.VISIBLE);
	                setData(view);
	            }
	
	            @Override
	            public void onAnimationEnd(Animation animation) {
	                view.clearAnimation();
	                final Animation anim = getAnim(false);
	                anim.setAnimationListener(new Animation.AnimationListener() {
	                    @Override
	                    public void onAnimationStart(Animation animation) {
	                        mHandler.postDelayed(new Runnable() {
	                            @Override
	                            public void run() {
	                                start2();
	                            }
	                        }, 800);
	                    }
	
	                    @Override
	                    public void onAnimationEnd(Animation animation) {
	                        view.clearAnimation();
	                        view.setVisibility(View.INVISIBLE);
	                    }
	
	                    @Override
	                    public void onAnimationRepeat(Animation animation) {
	
	                    }
	                });
	                mHandler.postDelayed(new Runnable() {
	                    @Override
	                    public void run() {
	                        view.startAnimation(anim);
	                    }
	                }, 2000);
	            }
	
	            @Override
	            public void onAnimationRepeat(Animation animation) {
	            }
	        });
	
	        view.startAnimation(anim);
	
	    }
	
	    private void start2() {
	        final View view = view2;
	
	        Animation anim = getAnim(true);
	        anim.setAnimationListener(new Animation.AnimationListener() {
	            @Override
	            public void onAnimationStart(Animation animation) {
	                view.setVisibility(View.VISIBLE);
	                setData(view);
	            }
	
	            @Override
	            public void onAnimationEnd(Animation animation) {
	                view.clearAnimation();
	                final Animation anim = getAnim(false);
	                anim.setAnimationListener(new Animation.AnimationListener() {
	                    @Override
	                    public void onAnimationStart(Animation animation) {
	                        mHandler.postDelayed(new Runnable() {
	                            @Override
	                            public void run() {
	                                start1();
	                            }
	                        }, 500);
	                    }
	
	                    @Override
	                    public void onAnimationEnd(Animation animation) {
	                        view.clearAnimation();
	                        view.setVisibility(View.INVISIBLE);
	                    }
	
	                    @Override
	                    public void onAnimationRepeat(Animation animation) {
	
	                    }
	                });
	                mHandler.postDelayed(new Runnable() {
	                    @Override
	                    public void run() {
	                        view.startAnimation(anim);
	                    }
	                }, 2000);
	            }
	
	            @Override
	            public void onAnimationRepeat(Animation animation) {
	            }
	        });
	
	        view.startAnimation(anim);
	
	    }
	
	    public Animation getAnim(boolean on) {
	        TranslateAnimation tran = new TranslateAnimation(
	                TranslateAnimation.RELATIVE_TO_SELF, 0,
	                TranslateAnimation.RELATIVE_TO_SELF, 0,
	                TranslateAnimation.RELATIVE_TO_SELF, on ? 1 : 0,
	                TranslateAnimation.RELATIVE_TO_SELF, on ? 0 : -1
	        );
	        tran.setFillAfter(true);
	        tran.setDuration(1500);
	        return tran;
	    }
	
	}  
```
要滚动的数据Bean：  
```java
	package com.gloomyer.toticedemo;
	
	/**
	 * Created by Gloomy on 16/6/23.
	 */
	public class Bean {
	    public String title;
	    public String value;
	
	    public Bean(String title, String value) {
	        this.title = title;
	        this.value = value;
	    }
	}  
```
Activity onCreate代码：  
```java
	protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	
	        notice = (NoticeVIew) findViewById(R.id.notice);
	
	        ArrayList<Bean> list = new ArrayList<>();
	
	        for (int i = 0; i < 10; i++) {
	            list.add(new Bean("title" + i, "value:" + i));
	        }
	
	        notice.setData(list);
	}
```