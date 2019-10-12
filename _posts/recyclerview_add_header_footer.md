---
title: RecyclerView添加头部和脚部
date: 2017-04-12 17:21:14
categories: Android
tags:
- recyclerview
- header&footer
---
<Excerpt in index | 首页摘要> 
> 给RecyclerView添加头部和脚部
>
> <!-- more -->
> <The rest of contents | 余下全文>  

#### 概述

项目里面一直没有运用到这方面的知识

今天看到这里了就决定搞个demo



先说下原理，

其实难度不大，如果看过ListView的源码就知道。

listview添加头部和脚部就是封装adapter来实现的。

那么我们可以抄袭listview来封装用户的adapter来实现这个添加头部和脚部



#### 实现代码

先模仿ListView

继承自RecyclerView

修改如下:

```java
package com.gloomyer.myapplication;

import android.content.Context;
import android.support.annotation.Nullable;
import android.support.v7.widget.RecyclerView;
import android.util.AttributeSet;
import android.view.View;

import java.util.ArrayList;

/**
 * 包装RecyclerView
 */

public class WrapRecyclerView extends RecyclerView {

    ArrayList<View> headers = new ArrayList<>();
    ArrayList<View> footers = new ArrayList<>();

    private RecyclerView.Adapter mAdapter;

    public WrapRecyclerView(Context context) {
        super(context);
    }

    public WrapRecyclerView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
    }

    public WrapRecyclerView(Context context, @Nullable AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }

    @Override
    public void setAdapter(Adapter adapter) {
        if (headers.isEmpty() && footers.isEmpty())
            mAdapter = adapter;
        else
            mAdapter = new WrapAdapter(headers, footers, adapter);
        super.setAdapter(mAdapter);
    }

    public void addHeaderView(View view) {
        headers.add(view);
        wrapAdapter();
    }

    public void removeHeaderView(View v) {
        headers.remove(v);
    }

    public void addFooterView(View v) {
        footers.add(v);
        wrapAdapter();
    }

    public void removeFooterView(View v) {
        footers.remove(v);
    }

    private void wrapAdapter() {
        if (mAdapter != null) {
            if (!(mAdapter instanceof WrapAdapter)) {
                setAdapter(new WrapAdapter(headers, footers, getAdapter()));
            }
        }
    }
}

```



这里的难度不大，就是如果用户添加了头部和脚部就封装adapter如果没有就不封装



具体看封装adapter

```java
package com.gloomyer.myapplication;

import android.support.v7.widget.RecyclerView;
import android.view.View;
import android.view.ViewGroup;

import java.util.ArrayList;

/**
 * 封装Adapter
 */

public class WrapAdapter extends RecyclerView.Adapter {
    public static final int HEADER_TYPE = Integer.MIN_VALUE;//默认的第一个头部的type
    public static final int FOOTER_TYPE = Integer.MAX_VALUE;//默认的第一个脚部的type

    private final RecyclerView.Adapter userAdapter;
    private final ArrayList<View> headers;
    private final ArrayList<View> footers;

    public WrapAdapter(ArrayList<View> headers, ArrayList<View> footers, RecyclerView.Adapter adapter) {
        this.userAdapter = adapter;
        this.headers = headers;
        this.footers = footers;
    }

    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        RecyclerView.ViewHolder holder;
        if (viewType >= HEADER_TYPE
                && viewType < (HEADER_TYPE + getHeadersCount())) {//判断如果在头部的type类型里面
            holder = new WrapHolder(headers.get(viewType - HEADER_TYPE));
        } else if (viewType <= FOOTER_TYPE
                && viewType > (FOOTER_TYPE - getFootersCount())) {//如果在脚部的type类型中
            holder = new WrapHolder(footers.get(FOOTER_TYPE - viewType));
        } else {
            holder = userAdapter.onCreateViewHolder(parent, viewType);//如果是用户adapter里面的
        }
        return holder;
    }

    @Override
    public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
        int numHeaders = getHeadersCount();
        if (position < numHeaders)//头部不作处理
            return;

        // Adapter
        final int adjPosition = position - numHeaders;
        if (userAdapter != null) {
            int adapterCount = userAdapter.getItemCount();
            if (adjPosition < adapterCount) {
                userAdapter.onBindViewHolder(holder, adjPosition);//用户adapter的逻辑 交还出去
            }
        }

        //HandlerFooter 脚部不需要处理
    }


    @Override
    public int getItemViewType(int position) {
        int numHeaders = getHeadersCount();
        if (position < numHeaders) {//头部，从Interger最小值开始，依次递增
            return HEADER_TYPE + position;
        }

        final int adjPosition = position - numHeaders;
        int adapterCount = userAdapter.getItemCount();
        if (userAdapter != null) {//正常用户adaptertype，注意调用的时候要记得避开头、脚的type值
            if (adjPosition < adapterCount) {
                return userAdapter.getItemViewType(adjPosition);
            }
        }

        return FOOTER_TYPE - (position - numHeaders - adapterCount);//脚部type，依次递减
    }


    @Override
    public int getItemCount() {
        if (userAdapter != null)
            return userAdapter.getItemCount() + getHeadersCount() + getFootersCount();
        return getHeadersCount() + getFootersCount();//至少显示顶部和脚部
    }

    public int getHeadersCount() {
        return headers.size();
    }

    public int getFootersCount() {
        return footers.size();
    }
	//封装需要，无所在意.
    private class WrapHolder extends RecyclerView.ViewHolder {

        public WrapHolder(View itemView) {
            super(itemView);
        }
    }
}

```

