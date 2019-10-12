---
title: Hook View OnClickListener 实现避免重复点击
date: 2017-5-25 09:45:51
categories: Android
tags: 
- Hook
- OnClickListener
- View
---
<Excerpt in index | 首页摘要> 
> Hook View OnClickListener 实现避免重复点击
><!-- more -->
><The rest of contents | 余下全文> 

#### 概述
做Android开发，经常碰到测试反馈一个问题。  
  
重复点击xx，打开(执行了)2次操作。  
  
自然这种情况是需要一定的避免的。如何避免呢？  
  
我们可以通过Rx来设置点击事件的方式，增加重复点击时间。  
  
我们可以通过封装OnCLickListener，每次设置点击事件，用我们的代理OnCLickListener(内部封装点击时间验证)来完成。  
  
当然，还有这种方式，HookView的OnclickListener来实现。  
（弊端，如果是类似于网络请求之后再设置的点击事件，这种具有延时的设置，可能会失效）  
  
#### 原理
View所有的事件都放在mListenerInfo这个成员变量里面  
  
通过反射获取View身上的getListenerInfo()方法，可以获取mListenerInfo对象  
  
然后遍历所有的View  
  
遍历每个View身上的mListenerInfo，找到OnClickListener。  
  
通过代理模式，创建一个我们自己的增加了重复点击判断的OnCliCkListenerProxy  
  
然后替换原来的。
#### 代码实现
Rx、代理这里就不具体展示了。  
  
先看代理OnClickListener  
  
```java
package com.microcity.microbus_go.listeners;

import android.view.View;

import java.util.Calendar;

/**
 * 代理onClickListener
 * 用于Hook View
 * 防止重复点击的问题
 */

public class OnClickListenerProxy implements View.OnClickListener {

    private static final int MIN_CLICK_DELAY_TIME = 500;
    private long lastClickTime = 0;

    public View.OnClickListener target;

    public OnClickListenerProxy(View.OnClickListener target) {
        this.target = target;
    }

    @Override
    public void onClick(View v) {
        long currentTime = Calendar.getInstance().getTimeInMillis();
        if (currentTime - lastClickTime > MIN_CLICK_DELAY_TIME) {
            lastClickTime = currentTime;
            if (target != null) target.onClick(v);
        }
    }
}

```
  
这个没有什么太大的难度，很简单。就是增加了一个时间的验证。  
  
然后创建一个HookViewUtils类  
  
这个类负责找到所有的View，然后反射修改OnCLickListener。
  
代码如下：  
```java
package com.microcity.microbus_go.utils;

import android.app.Activity;
import android.view.View;
import android.view.ViewGroup;

import com.microcity.microbus_go.listeners.OnClickListenerProxy;

import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.List;

/**
 * Hook utils
 * 利用Hook技术
 * hook View的onClickListener
 * 然后通过代理onClickListener
 * 避免了重复点击的问题
 *
 * @author Gloomy
 * @version 1.0
 * @date 2017年5月24日
 */

public class HookViewUtils {
    public static void hook(Activity mAct) {
        try {
            Class viewClazz = Class.forName("android.view.View");
            //事件监听器都是这个实例保存的
            Method listenerInfoMethod = viewClazz.getDeclaredMethod("getListenerInfo");
            if (!listenerInfoMethod.isAccessible()) {
                listenerInfoMethod.setAccessible(true);
            }

            List<View> list = getAllChildViews(mAct.getWindow().getDecorView());

            for (View view : list) {
                Object listenerInfoObj = listenerInfoMethod.invoke(view);

                Class listenerInfoClazz = Class.forName("android.view.View$ListenerInfo");

                Field onClickListenerField = listenerInfoClazz.getDeclaredField("mOnClickListener");

                if (!onClickListenerField.isAccessible()) {
                    onClickListenerField.setAccessible(true);
                }
                View.OnClickListener mOnClickListener = (View.OnClickListener) onClickListenerField.get(listenerInfoObj);
                //自定义代理事件监听器
                View.OnClickListener onClickListenerProxy = new OnClickListenerProxy(mOnClickListener);
                //更换
                onClickListenerField.set(listenerInfoObj, onClickListenerProxy);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }


    private static List<View> getAllChildViews(View view) {
        List<View> allchildren = new ArrayList<View>();
        if (view instanceof ViewGroup) {
            ViewGroup vp = (ViewGroup) view;
            for (int i = 0; i < vp.getChildCount(); i++) {
                View viewchild = vp.getChildAt(i);
                allchildren.add(viewchild);
                //再次 调用本身（递归）
                allchildren.addAll(getAllChildViews(viewchild));
            }
        }
        return allchildren;
    }

}

```
  
第三步:  
  
找到你的BaseActivity  
在其onCreate中加入:  
```java  
getWindow().getDecorView().postDelayed(() -> { HookViewUtils.hook(this) }, 300);
```

至此，就大功告成了，快去试试看吧~