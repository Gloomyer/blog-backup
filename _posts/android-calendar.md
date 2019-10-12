---
title: Android自己实现一个日历 
date: 2016-9-20 11:03:45
categories: Android
tags: 
- Android
- 自定义View
- 日历
---
<Excerpt in index | 首页摘要> 
>Android自己实现一个日历 
<!-- more -->
<The rest of contents | 余下全文> 
  
##  概述  ##
项目要做类似于一个心路历程的日期展示.  
  
效果图:
<img src="http://gloomyer.com/img/img/android-calendar.jpg" width = "360" height = "640" align=center />

##  分析  ##
因为顶部的图要做放大方法的效果.  
  
所以决定将从年月的位置开始到底部封装成一个View.  
  
##  实现  ##
DateShowView代码:
```java
package com.qingxiang.ui.view;

import android.content.Context;
import android.support.v7.widget.GridLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.util.AttributeSet;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.FrameLayout;
import android.widget.LinearLayout;
import android.widget.TextView;

import com.qingxiang.ui.R;
import com.qingxiang.ui.bean.DateShowBean;
import com.qingxiang.ui.common.CommonViewHolder;
import com.qingxiang.ui.utils.Dateutils;
import com.qingxiang.ui.utils.DensityUtils;

import java.util.List;

/**
 * 日期显示View
 *
 * @author Gloomy
 */
public class DateShowView extends FrameLayout implements View.OnClickListener {
    private TextView tvTitle;
    private RecyclerView rv;
    private MyAdapter mAdapter;
    private List<DateShowBean> days;
    private int year;
    private int month;
    private int day;
    private String selectTime;

    public DateShowView(Context context) {
        this(context, null);
    }

    public DateShowView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public DateShowView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    //初始化
    private void init() {
        year = Dateutils.getYear();
        month = Dateutils.getMonth();
        day = Dateutils.getDay();
        selectTime = Dateutils.getNowDate();

        View.inflate(getContext(), R.layout.view_date_show_view, this);
        tvTitle = (TextView) findViewById(R.id.tv_title);
        rv = (RecyclerView) findViewById(R.id.id_rv);
        findViewById(R.id.iv_left).setOnClickListener(this);
        findViewById(R.id.iv_right).setOnClickListener(this);

        GridLayoutManager gridLayoutManager = new GridLayoutManager(getContext(), 7);
        rv.setLayoutManager(gridLayoutManager);
        mAdapter = new MyAdapter();
        rv.setAdapter(mAdapter);

        tvTitle.setText(selectTime);
        days = Dateutils.getDays(selectTime);
		//说一下这里，因为涉及到滚动View嵌套滚动View，所以自己写了一个改变刷新，因为要手动控制View宽高.
        mAdapter.changedData();
    }

    @Override
    public void onClick(View v) {
        String[] split = selectTime.split("\\.");
        int y = Integer.parseInt(split[0]);
        int m = Integer.parseInt(split[1]);
        switch (v.getId()) {
            case R.id.iv_left: {
                m--;
                if (m <= 0) {
                    m = 12;
                    y--;
                }
                break;
            }
            case R.id.iv_right: {
                m++;
                if (m >= 13) {
                    m = 1;
                    y++;
                }
                break;
            }
        }
        selectTime = y + "." + (m <= 9 ? "0" : "") + m;
        tvTitle.setText(selectTime);
        days = Dateutils.getDays(selectTime);
        mAdapter.changedData();
    }

    private class MyAdapter extends RecyclerView.Adapter<CommonViewHolder> {

        @Override
        public CommonViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
            LayoutInflater inflater = LayoutInflater.from(getContext());
            View view = inflater.inflate(R.layout.item_alarm_day, parent, false);
            return new CommonViewHolder(view);
        }

        @Override
        public void onBindViewHolder(CommonViewHolder holder, int position) {
            DateShowBean day = days.get(position);
            if (day == null) {
                holder.getContentView().setVisibility(INVISIBLE);
            } else {
                holder.getContentView().setVisibility(VISIBLE);
                holder.setText(R.id.item_tv, day.nowDay);
                boolean isNow = false;
                String[] dateStr = selectTime.split("\\.");

                int y = Integer.parseInt(dateStr[0]);
                int m = Integer.parseInt(dateStr[1]);

                if (y == year && m == month) {
                    int d = Integer.parseInt(day.nowDay);
                    if (d == DateShowView.this.day) {
                        isNow = true;
                    }
                }


                if (isNow)
                    holder.getV(R.id.item_point).setVisibility(View.VISIBLE);
                else
                    holder.getV(R.id.item_point).setVisibility(View.INVISIBLE);
            }
        }

        @Override
        public int getItemCount() {
            return days == null ? 0 : days.size();
        }

        public void changedData() {
            int count = getItemCount();
            int line = count / 7;
            if (count % 7 > 0)
                line++;
            LinearLayout.LayoutParams layoutParams = (LinearLayout.LayoutParams) rv.getLayoutParams();
            layoutParams.height = line * DensityUtils.dp2px(getContext(), 34);
            rv.setLayoutParams(layoutParams);
            notifyDataSetChanged();
        }
    }

}

```
view_date_show_view的布局代码:
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/alpha"
    android:orientation="vertical">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="48dp"
        android:background="@color/alpha">

        <ImageView
            android:id="@+id/iv_left"
            android:layout_width="48dp"
            android:layout_height="48dp"
            android:layout_marginLeft="10dp"
            android:background="@color/alpha"
            android:padding="16dp"
            android:src="@mipmap/icon_left" />

        <ImageView
            android:id="@+id/iv_right"
            android:layout_width="48dp"
            android:layout_height="48dp"
            android:layout_gravity="right"
            android:layout_marginRight="10dp"
            android:background="@color/alpha"
            android:padding="16dp"
            android:src="@mipmap/icon_right" />

        <TextView
            android:id="@+id/tv_title"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:background="@color/alpha"
            android:text="2016.02"
            android:textColor="@color/gray"
            android:textSize="18sp" />
    </FrameLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="40dp"
        android:background="@color/alpha"
        android:orientation="horizontal">

        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:background="@color/alpha"
            android:gravity="center"
            android:text="日"
            android:textColor="@color/gray"
            android:textSize="14sp" />

        <TextView
            android:id="@+id/textView2"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:background="@color/alpha"
            android:gravity="center"
            android:text="一"
            android:textColor="@color/gray"
            android:textSize="14sp" />

        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:background="@color/alpha"
            android:gravity="center"
            android:text="二"
            android:textColor="@color/gray"
            android:textSize="14sp" />

        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:background="@color/alpha"
            android:gravity="center"
            android:text="三"
            android:textColor="@color/gray"
            android:textSize="14sp" />

        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:background="@color/alpha"
            android:gravity="center"
            android:text="四"
            android:textColor="@color/gray"
            android:textSize="14sp" />

        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:background="@color/alpha"
            android:gravity="center"
            android:text="五"
            android:textColor="@color/gray"
            android:textSize="14sp" />

        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:background="@color/alpha"
            android:gravity="center"
            android:text="六"
            android:textColor="@color/gray"
            android:textSize="14sp" />
    </LinearLayout>

    <android.support.v7.widget.RecyclerView
        android:id="@+id/id_rv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="12dp"
        android:background="@color/alpha" />
</LinearLayout>
```
日期的实际操作工具类DateUtils代码:
```java
package com.qingxiang.ui.utils;

import com.qingxiang.ui.bean.DateShowBean;

import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

/**
 * 日期的一些操作工具类
 * 配合DateShowView使用!
 *
 * @author Gloomy
 */
public class Dateutils {
    /**
     * 获取当前年月
     * 格式为:xxxx.xx
     *
     * @return
     */
    public static String getNowDate() {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy.MM");
        return sdf.format(new Date(System.currentTimeMillis()));
    }

    /**
     * 获取现在的年
     *
     * @return
     */
    public static int getYear() {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy");
        return Integer.parseInt(sdf.format(new Date(System.currentTimeMillis())));
    }

    public static int getMonth(){
        SimpleDateFormat sdf = new SimpleDateFormat("MM");
        return Integer.parseInt(sdf.format(new Date(System.currentTimeMillis())));
    }

    public static int getDay(){
        SimpleDateFormat sdf = new SimpleDateFormat("dd");
        return Integer.parseInt(sdf.format(new Date(System.currentTimeMillis())));
    }

    /**
     * 根据年月获取月内具体日期
     *
     * @param date 要获取具体日期的年月，格式为yyyy.MM
     * @return 返回数据如果1号不在星期日，前面将会用null填充,eg:2016.9 有30天，1号在星期四，那么将会返回33长度的List，前3个为null
     */
    public static List<DateShowBean> getDays(String date) {
        List<DateShowBean> list = new ArrayList<>();

        String[] split = date.split("\\.");
        int y = Integer.parseInt(split[0]);
        int m = Integer.parseInt(split[1]);

        /**
         * 计算这个月有多少天
         */
        int daySize = 0;
        switch (m) {
            case 1:
            case 3:
            case 5:
            case 7:
            case 8:
            case 10:
            case 12:
                daySize = 31;
                break;
            case 4:
            case 6:
            case 9:
            case 11:
                daySize = 30;
                break;
            case 2:
                boolean isR = ((y % 4 == 0 && y % 100 != 0) || y % 400 == 0);
                if (isR)
                    daySize = 29;
                else
                    daySize = 28;
                break;
        }


        //基姆拉尔森计算公式
        //W= (d+2*m+3*(m+1)/5+y+y/4-y/100+y/400) mod 7
        //在公式中d表示日期中的日数，m表示月份数，y表示年数
        //注意：在公式中有个与其他公式不同的地方：
        //把一月和二月看成是上一年的十三月和十四月，例：如果是2004-1-10则换算成：2003-13-10来代入公式计算。
        if (m <= 2) {
            y--;
            m = 12 + m;
        }

        int w = (1 + 2 * m + 3 * (m + 1) / 5 + y + y / 4 - y / 100 + y / 400) % 7;

        /**
         * 计算1号为星期几，并且计算补位多少个
         */
        int nSize = w + 1;
        if (nSize == 7)
            nSize = 0;
        for (int i = 0; i < nSize; i++) {
            list.add(null);
        }
        /**
         * 依次填充后续日期
         */
        for (int i = 0; i < daySize; i++) {
            DateShowBean d = new DateShowBean();
            d.nowDay = "" + (i + 1);
            d.week = w;
            list.add(d);
            w++;
            if (w > 6)
                w = 0;
        }

        return list;
    }
}

```