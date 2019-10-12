---
title: 模仿Tablayout效果
date: 2016-8-26 16:32:30
categories: Android
tags:
- Android
- 自定义View
---
<Excerpt in index | 首页摘要> 
> 模仿Tablayout效果 
>
<!-- more -->
<The rest of contents | 余下全文>  
  
![](http://www.gloomyer.com/img/img/tablayout-demo.gif) 
  
项目需要用到这个效果.  
  
谷歌提供了一种方案Tablayout(兼容包)的组件，可以非常方便的实现这个效果.  
  
苛刻的产品，要求文字与红点距离必须为0,文字距离底部的线距离必须是14dp.  
  
那么问题就来了.谷歌原生的上下距离是相等的，内部的TextView是居中显示的.而且我们又控制不了.  
  
没办法，只有自己完全写一个了.直接来看代码吧.  
```java
	package com.qingxiang.ui.view;
	
	import android.content.Context;
	import android.support.v4.view.PagerAdapter;
	import android.support.v4.view.ViewPager;
	import android.util.AttributeSet;
	import android.util.TypedValue;
	import android.view.Gravity;
	import android.view.View;
	import android.view.ViewGroup;
	import android.widget.FrameLayout;
	import android.widget.LinearLayout;
	import android.widget.TextView;
	
	import com.qingxiang.ui.R;
	import com.qingxiang.ui.utils.DensityUtils;
	import com.qingxiang.ui.utils.Utils;
	
	/**
	 * 顶部滑动注入组件
	 */
	public class TabInject extends FrameLayout implements ViewPager.OnPageChangeListener {
	
	    private static final String TAG = "TabInject";
	    private LinearLayout continar;
	    private PagerAdapter mAdapter;
	    private View line;
	    private int curPage;
	    private ViewPager mViewPager;
	    private TextView lastSelectChild;
	    private int lineWidth;
	
	    public TabInject(Context context) {
	        this(context, null);
	    }
	
	    public TabInject(Context context, AttributeSet attrs) {
	        this(context, attrs, 0);
	    }
	
	    public TabInject(Context context, AttributeSet attrs, int defStyleAttr) {
	        super(context, attrs, defStyleAttr);
	        init();
	    }
	
	    private void init() {
			//创建顶部textview的容器
	        continar = new LinearLayout(getContext());
	        addView(continar, new ViewGroup.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT,
	                ViewGroup.LayoutParams.MATCH_PARENT));
	    }
	
	    public void setAdapter(ViewPager vp, PagerAdapter adapter) {
	        mAdapter = adapter;
	        mViewPager = vp;
	        curPage = mViewPager.getCurrentItem();
	        int count = mAdapter.getCount();
			//根据屏幕宽度与头部count来计算底部指示线的宽度
	        lineWidth = Utils.getScreenWidth() / count;
	        int lineHeight = DensityUtils.dp2px(getContext(), 2);
	
	        line = new View(getContext());
	        line.setBackgroundColor(getResources().getColor(R.color.colorPrimary));
	        FrameLayout.LayoutParams lineParams
	                = new FrameLayout.LayoutParams(lineWidth, lineHeight);
	        lineParams.gravity = Gravity.BOTTOM;
	        addView(line, lineParams);
	
	        //给顶部增加TextView
	        for (int i = 0; i < count; i++) {
	            TextView tv = new TextView(getContext());
	            LinearLayout.LayoutParams
	                    params = new LinearLayout.LayoutParams(0, LinearLayout.LayoutParams.MATCH_PARENT);
	            params.weight = 1;
	            tv.setGravity(Gravity.TOP | Gravity.CENTER_HORIZONTAL);
	            tv.setText(mAdapter.getPageTitle(i));
	
	            if (curPage == i) {
	                tv.setTextColor(getResources().getColor(R.color.colorPrimary));
	                lastSelectChild = tv;
	            } else
	                tv.setTextColor(getResources().getColor(R.color.middleDark));
	
	            tv.setBackgroundColor(getResources().getColor(R.color.alpha));
	            tv.setTextSize(TypedValue.COMPLEX_UNIT_SP, 14);
	
	            final int finalI = i;
	            tv.setOnClickListener(v -> {
	                mViewPager.setCurrentItem(finalI);
	            });
	            continar.addView(tv, params);
	        }
	        mViewPager.addOnPageChangeListener(this);
	    }
	
	    @Override
	    protected void onDetachedFromWindow() {
	        super.onDetachedFromWindow();
	        mViewPager.removeOnPageChangeListener(this);
	    }
	
	    @Override
	    public void onPageScrolled(int pos, float off, int offPx) {
			//通过用户移动的offset值修改指示线的leftMargin.达到移动指示线的目的
	        int moveX = (int) (lineWidth * off);
	        LayoutParams layoutParams = (LayoutParams) line.getLayoutParams();
	        layoutParams.leftMargin = (pos * lineWidth) + moveX;
	
	        int dX = layoutParams.leftMargin - (pos * lineWidth);
	
	        if (dX > 0 && dX <= 1) {
	            layoutParams.leftMargin = (pos * lineWidth);
	        }
	
	        if (dX < lineWidth && dX >= (lineWidth - 1)) {
	            layoutParams.leftMargin = (pos + 1) * lineWidth;
	        }
	
	        line.setLayoutParams(layoutParams);
	    }
	
	    @Override
	    public void onPageSelected(int position) {
	        /**
	         * 更改之后修正被选择标签的文字颜色
	         */
	        TextView tv = (TextView) continar.getChildAt(position);
	        lastSelectChild.setTextColor(getResources().getColor(R.color.middleDark));
	        tv.setTextColor(getResources().getColor(R.color.colorPrimary));
	
			//即便不是滑动移动ViewPager，也会触发onPageScrolled方法，所以这里的动画移动就没必要做了
	        /*{//移动底部的指示线
	            //首先获取现在线的位置
	            int lineFromX = curPage * lineWidth;
	            //获取将要到达的位置
	            int lineToX = position * lineWidth;
	            FrameLayout.LayoutParams layoutParams = (LayoutParams) line.getLayoutParams();
	
	            if (Math.abs(lineToX - layoutParams.leftMargin) < DensityUtils.dp2px(getContext(), 5)) {
	                layoutParams.leftMargin = lineToX;
	                TranslateAnimation translate =
	                        new TranslateAnimation(
	                                TranslateAnimation.RELATIVE_TO_SELF, 0,
	                                TranslateAnimation.RELATIVE_TO_SELF, position - curPage,
	                                TranslateAnimation.RELATIVE_TO_SELF, 0,
	                                TranslateAnimation.RELATIVE_TO_SELF, 0
	                        );
	                translate.setDuration(200);
	                translate.setFillAfter(true);
	                translate.setAnimationListener(new AnimationListener() {
	                    @Override
	                    public void onAnimationEnd(Animation animation) {
	                        line.setLayoutParams(layoutParams);
	                        line.clearAnimation();
	                    }
	                });
	                line.startAnimation(translate);
	            } else {
	                layoutParams.leftMargin = lineToX;
	                line.setLayoutParams(layoutParams);
	            }
	        }*/
	
	        lastSelectChild = tv;
	        curPage = position;
	    }
	
	    @Override
	    public void onPageScrollStateChanged(int state) {
	
	    }
	}
```