---
title: Android拖拽排序功能实现
date: 2016-9-10 11:15:10
categories: Android
tags:
- Android
- DragSort
- 拖拽排序
- 自定义View
---
<Excerpt in index | 首页摘要> 
> Android拖拽排序功能实现  
>
<!-- more -->
<The rest of contents | 余下全文>  
  
先来看一下效果:  
  
![效果图](http://gloomyer.com/img/img/android-drag-sort.gif)  
  
##思路
Android本身就给我们提供了拖拽的实现.  
  
在View身上都有一个startDrag()方法.  
  
想要实现拖拽效果，还需要让拖拽的父View实现一个onDragListener.  
  
##实现概述  
  
首先，我选择继承自GradLayout,然后手动创建child.  
  
然后在每个item身上添加长按事件.  
  
长按时间内容为启动拖拽.  
```java
 	v.startDrag(null, new DragShadowBuilder(v), null, 0);
```
new DragShadowBuilder(view)是创建一个用于拖拽的虚影,它会自动跟随手势移动.  
  
然后在父容器实现OnDragListener,在开始拖拽的时候实现获取每个View的坐标，放置在集合中.  
  
然后在手势移动的时候，判断当前手势的坐标点是否在某个item上，如果在就移除当前要拖拽的view，然后添加到获取到的坐标点上.  
  
##具体代码实现  
```java
	package com.qingxiang.ui.view;
	
	import android.animation.LayoutTransition;
	import android.content.Context;
	import android.graphics.Rect;
	import android.util.AttributeSet;
	import android.view.DragEvent;
	import android.view.LayoutInflater;
	import android.view.View;
	import android.widget.GridLayout;
	
	import com.bumptech.glide.Glide;
	import com.qingxiang.ui.R;
	import com.qingxiang.ui.bean.TagGroupBean;
	import com.qingxiang.ui.common.CommonViewHolder;
	import com.qingxiang.ui.utils.DensityUtils;
	import com.qingxiang.ui.utils.L;
	import com.qingxiang.ui.utils.QNUtils;
	import com.qingxiang.ui.utils.Utils;
	
	import java.util.ArrayList;
	import java.util.List;
	
	/**
	 * 拖拽排序View
	 */
	public class DragSortView extends GridLayout {
	    private static final String TAG = "DragSortView";
	    private static final int COLUMN_COUNT = 4;
	    private int itemSize;
	    private View dragView;
	    private List<Rect> viewRects;
	    private int margin;
	
	    public DragSortView(Context context) {
	        this(context, null);
	    }
	
	    public DragSortView(Context context, AttributeSet attrs) {
	        this(context, attrs, 0);
	    }
	
	    public DragSortView(Context context, AttributeSet attrs, int defStyleAttr) {
	        super(context, attrs, defStyleAttr);
	        init();
	    }
	
	    private void init() {
	        //设置一行多少个条目
	        setColumnCount(COLUMN_COUNT);
	        //child入场退场动画
	        LayoutTransition transition = new LayoutTransition();
	        transition.setDuration(150);
	        setLayoutTransition(transition);
	        //计算item 高度和宽度
	        int width = Utils.getScreenWidth();//屏幕高度
	        width -= DensityUtils.dp2px(getContext(), 20 + ((COLUMN_COUNT - 1) * 3));//y轴item布局总数
	        itemSize = (int) (width * 1.0f / COLUMN_COUNT + 0.5f); //item宽度,高度都是他
	        //初始化数据
	        viewRects = new ArrayList<>();
	        margin = DensityUtils.dp2px(getContext(), 3);
	    }
	
	    public void setData(List<TagGroupBean> mDatas) {
	        removeAllViews();
	        int i = 0;
	        for (TagGroupBean mData : mDatas) {
	            addView(i, mData);
	            i++;
	        }
	        //实现拖拽
	        setOnDragListener(new DragListener());
	    }
	
	    private void addView(int pos, TagGroupBean mData) {
	        //创建item
	        LayoutInflater inflater = LayoutInflater.from(getContext());
	        View view = inflater.inflate(R.layout.item_find_edit_sub, this, false);
	        CommonViewHolder holder = new CommonViewHolder(view);
	        //设置文字
	        holder.getTextView(R.id.item_tv).setText(mData.getTag());
	        //加载图片
	        Glide.with(getContext())
	                .load(QNUtils.formatUrl(mData.getIcon(), 1, 400, 400, false))
	                .placeholder(R.mipmap.area_img1)
	                .into(holder.getIv(R.id.item_iv));
	        //长按
	        holder.getContentView().setOnLongClickListener(v -> {
	            dragView = v;
	            v.startDrag(null, new DragShadowBuilder(v), null, 0);
	            return true;
	        });
	        addView(view, getParams(pos));
	    }
	
	
	    private class DragListener implements OnDragListener {
	        /**
	         * 该值存放的是上一次移动到的位置
	         */
	        int lastIndex = -1;
	
	        @Override
	        public boolean onDrag(View v, DragEvent event) {
	            switch (event.getAction()) {
	                case DragEvent.ACTION_DRAG_STARTED:
	                    initRects();
	                    lastIndex = indexOfChild(dragView);
	                    if (dragView != null)
	                        dragView.setEnabled(false);
	                    return true;
	                case DragEvent.ACTION_DRAG_ENDED:
	                    lastIndex = -1;
	                    if (dragView != null) {
	                        dragView.setEnabled(true);
	                        dragView = null;
	                    }
	                    return true;
	                case DragEvent.ACTION_DRAG_LOCATION:
	                    int moveX = (int) event.getX();
	                    int moveY = (int) event.getY();
	                    L.i(TAG, "location X:" + moveX + " Y:" + moveY);
	                    int findIndex = findTouchChildIndex(moveX, moveY);
	                    if (findIndex != -1 && findIndex != lastIndex && dragView != null) {
	                        removeView(dragView);
	                        addView(dragView, findIndex);
	                        lastIndex = findIndex;
	                    }
	                    return true;
	            }
	            return false;
	        }
	    }
	
		/**
		 * 获取当前所有child的坐标区域
		 */
	    private void initRects() {
	        viewRects.clear();
	        for (int i = 0; i < getChildCount(); i++) {
	            View child = getChildAt(i);
	            Rect object = new Rect(child.getLeft(), child.getTop(), child.getRight(), child.getBottom());
	            viewRects.add(object);
	        }
	    }
	
		/**
		  找到当前坐标所在的pos点，成功返回pos，失败返回-1*
		 */
	    private int findTouchChildIndex(int moveX, int moveY) {
	        for (int i = 0; i < viewRects.size(); i++) {
	            boolean isFind = viewRects.get(i).contains(moveX, moveY);
	            if (isFind) {
	                return i;
	            }
	        }
	        return -1;
	    }
	
		/**
		 * 获取布局参数
		 */
	    private GridLayout.LayoutParams getParams(int pos) {
	        GridLayout.LayoutParams params = new GridLayout.LayoutParams();
	
	        params.height = itemSize;
	        params.width = itemSize;
	        params.leftMargin = margin;
	
	        if (pos < 4) {
	            params.bottomMargin = margin;
	        }
	
	        return params;
	    }
	}
```