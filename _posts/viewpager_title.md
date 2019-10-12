---
title: 自定义View
date: 2017-11-01 15:46:00
categories: Android
tags: 
- 自定义View
---
<Excerpt in index | 首页摘要> 
> 自定义View
> <!-- more -->
> <The rest of contents | 余下全文> 

##  展示  ##
项目需要实现这么个效果

<iframe width="1024" height="768" src="https://gloomyer.com/upload/Screenrecorder-2017-11-01-15-59-55-343.mp4" frameborder="0" allowfullscreen></iframe>

##  思路  ##
Activity撤销SystemStatus.把整个app顶到状态栏部分

注意，属性是v21以上才支持 所以建立在value-v21的styles下
```
<style name="MainTheme" parent="AppTheme">
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
        <item name="colorPrimary">@android:color/transparent</item>
        <item name="colorPrimaryDark">@android:color/transparent</item>
        <item name="android:windowTranslucentStatus">true</item>
        <item name="android:statusBarColor">@android:color/transparent</item>
</style>
```

然后在几个Fragment的上一层抽一个base加入如下代码:
```
  @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        ViewGroup view = (ViewGroup) super.onCreateView(inflater, container, savedInstanceState);
        int actionBarSize = DensityUtil.getActionBarHeight();
        int size = 0;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
            size = DensityUtil.getSystemStatusHeight();//这个方法就是读取系统状态栏高度 百度一下即可
        }
        view.setPadding(view.getPaddingLeft(), view.getPaddingTop() + actionBarSize + size, view.getPaddingRight(), view.getPaddingBottom());
        return view;
    }
```

##  代码  ##
```
package com.feparty.tbh.view;

import android.content.Context;
import android.support.annotation.NonNull;
import android.support.annotation.Nullable;
import android.support.v4.view.ViewPager;
import android.util.AttributeSet;
import android.util.TypedValue;
import android.view.Gravity;
import android.view.View;
import android.view.ViewGroup;
import android.widget.FrameLayout;
import android.widget.TextView;

import com.feparty.tbh.R;
import com.feparty.tbh.common.Loger;
import com.feparty.tbh.utils.DensityUtil;

/**
 * Created by gloomy on 17-10-31.
 */

public class MyTabLayout extends FrameLayout implements ViewPager.OnPageChangeListener {

    private static final String TAG = "MyTabLayout";
    private static final String[] titles = {"+好友", "圈子", "消息", "投票", "我", "设置"};

    private ViewPager mViewPager;
    private TextView[] tabs;
    private int height;
    private int width;
    private int tabWidth;
    private int centerMargin;
    private int leftMargin;
    private int rightMargin;
    private int currenItem; //当前的位置
    private int moveSize;

    public MyTabLayout(@NonNull Context context) {
        this(context, null);
    }

    public MyTabLayout(@NonNull Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public MyTabLayout(@NonNull Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);

        //有6个tab
        float size = getContext().getResources().getDimension(R.dimen.titleSize);
        tabWidth = (int) (size * 3);

        tabs = new TextView[titles.length];
        for (int i = 0; i < titles.length; i++) {
            tabs[i] = new TextView(context);
            tabs[i].setText(titles[i]);
            tabs[i].setTextSize(TypedValue.COMPLEX_UNIT_PX, size);
            tabs[i].setGravity(Gravity.CENTER);
            FrameLayout.LayoutParams layoutParams = new FrameLayout.LayoutParams(tabWidth, ViewGroup.LayoutParams.MATCH_PARENT);
            addView(tabs[i], layoutParams);
            tabs[i].setVisibility(GONE);
        }
    }

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        width = DensityUtil.getScreenWidth();
        height = DensityUtil.getActionBarHeight();
        Loger.E(TAG, "width:" + width + "---height:" + height);

        //中间点 左边 右边的leftMargin值:
        centerMargin = (width / 2) - (tabWidth / 2);
        leftMargin = DensityUtil.dip2px(16);
        rightMargin = width - DensityUtil.dip2px(16) - tabWidth;


        //move size
        moveSize = centerMargin - leftMargin;

    }

    /**
     * 设置当前显示的是第几个
     *
     * @param currenItem
     */
    public void setCurrentItem(int currenItem) {
        this.currenItem = currenItem;

        tabs[currenItem].setVisibility(VISIBLE);
        LayoutParams layoutParams = (LayoutParams) tabs[currenItem].getLayoutParams();
        layoutParams.leftMargin = centerMargin;

        if (currenItem > 0) {
            //存在左边
            tabs[currenItem - 1].setVisibility(VISIBLE);
            tabs[currenItem - 1].setAlpha(0.5f);
            layoutParams = (LayoutParams) tabs[currenItem - 1].getLayoutParams();
            layoutParams.leftMargin = leftMargin;
        }


        if (currenItem <= titles.length - 2) {
            //存在右边
            tabs[currenItem + 1].setVisibility(VISIBLE);
            tabs[currenItem + 1].setAlpha(0.5f);
            layoutParams = (LayoutParams) tabs[currenItem + 1].getLayoutParams();
            layoutParams.leftMargin = rightMargin;
        }
    }

    /**
     * 设置绑定的ViewPager
     *
     * @param mViewPager
     */
    public void setViewPager(ViewPager mViewPager) {
        this.mViewPager = mViewPager;
        for (int i = 0; i < titles.length; i++) {
            int finalI = i;
            tabs[i].setOnClickListener(v -> {
                mViewPager.setCurrentItem(finalI);
            });
        }
        postDelayed(() -> {
            mViewPager.addOnPageChangeListener(this);
        }, 1000);
    }

    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
        Loger.E(TAG, "scroll " + ",position:" + position + "|offset:" + positionOffset + "|offsetpx:" + positionOffsetPixels);
        TextView tab1 = tabs[position];
        tab1.setVisibility(VISIBLE);
        LayoutParams params = (LayoutParams) tab1.getLayoutParams();
        params.leftMargin = (int) (leftMargin + (moveSize * (1 - positionOffset)));
        tab1.setAlpha(.5f + (.5f * (1.0f - positionOffset)));
        tab1.setLayoutParams(params);

        if (position < titles.length - 1) {
            //存在右边的
            TextView tab2 = tabs[position + 1];
            tab2.setVisibility(VISIBLE);
            params = (LayoutParams) tab2.getLayoutParams();
            params.leftMargin = (int) (rightMargin - (moveSize * positionOffset));
            tab2.setAlpha(.5f + (.5f * positionOffset));
            tab2.setLayoutParams(params);
        }

        if (position > 0) {
            //存在左边的
            TextView tab3 = tabs[position - 1];
            tab3.setVisibility(VISIBLE);
            params = (LayoutParams) tab3.getLayoutParams();
            params.leftMargin = (int) (leftMargin - (moveSize * positionOffset));
            Loger.E(TAG, "left:" + params.leftMargin);
            if (params.leftMargin >= -tabWidth) {
                tab3.setVisibility(VISIBLE);
            } else {
                tab3.setVisibility(GONE);
            }
            tab3.setAlpha(.5f * (1.0f - positionOffset));
            tab3.setLayoutParams(params);
        }

        if (position < titles.length - 2) {
            //存在最右边的
            TextView tab4 = tabs[position + 2];
            tab4.setVisibility(VISIBLE);
            params = (LayoutParams) tab4.getLayoutParams();
            params.leftMargin = (int) (rightMargin + (moveSize * (1 - positionOffset)));
            if (params.leftMargin <= width + tabWidth) {
                tab4.setVisibility(VISIBLE);
            } else {
                tab4.setVisibility(GONE);
            }
            tab4.setAlpha(.5f * positionOffset);
            tab4.setLayoutParams(params);
        }
    }


    @Override
    public void onPageSelected(int position) {
        currenItem = position;
    }

    @Override
    public void onPageScrollStateChanged(int state) {
    }


    public void onDestroy() {
        mViewPager.removeOnPageChangeListener(this);
        mViewPager = null;
    }
}

```
