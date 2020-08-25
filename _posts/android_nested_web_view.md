---
title: Android 滚动 View 嵌套 WebView
date: 2020-08-25 22:00:00
categories: Android
tags:
- Android
---
<Excerpt in index | 首页摘要>
> Android 在可滚动View中嵌套WebView
<!-- more -->
<The rest of contents | 余下全文>  

## 为啥这么做?
项目中商品详情顶部UI是利用原生渲染，底部详情为了运营方便编辑走的是H5页面

这种场景下 就需要在一个可以滚动的View中去嵌套WebView

最开始项目外部用的NestedScrollView 顶部利用了一个关闭了嵌套滚动的RecyclerView 实现 底部是webView

类似这样的结构
```xml
<NestedScrollView>
    <LinearLayout>
        <RecyclerView>
        </RecyclerView>
        <WebView>
        </WebView>
    </LinearLayout>
</NestedScrollView>
```

webview 底部商品详情是相对固定的

但是后续业务导致webview 底部又开始有推荐商品，是一个分页接口的列表。

造成了一个神奇的现象，webview高度无限增长

在chrome webview调试工具中看到 webview开始加载后高度不停的增高

不一会就变成了几万像素

## 解决方案1,适用于WebView加载的内容是固定的
直接替换NestedScrollView为ScrollView

如果webview 加载的内容需要滚动事件，此方法不可用.

## 解决方案2,适用于WebView加载的内容是固定的

这种方案其实非常简单，在WebView 加载完成之后获取内容高度 给webview 钉死高度

如果webview 加载的内容需要滚动事件，此方法不可用.

具体没实现 就不贴代码了

思路是webview有个方法是getContentHeight()可以获取网页内容高度

## 解决方案3/4

3/4的前提都一样，首先webview 不支持嵌套滚动，第一步是给webview 添加嵌套滚动

这是大佬写好的嵌套滚动WebView[NestedWebView](https://stackoverflow.com/questions/57654466/nestedwebview-working-properly-with-scrollingviewbehavior)

3/4的区别就是是否继续NestedScrollView

继续用要重写NestScrollView 去重写嵌套滚动的几个方法，来派发事件 实现类似吸顶效果

但是有个取巧的方案，那就是CoordinatorLayout

我用了取巧的方案，因为我觉得我自己重写事件派发也未必有CoordinatorLayout写的好，不如直接用好了。

最后的项目结构就是:

记得关闭RecyclerView的嵌套滚动哟.

```xml
<androidx.coordinatorlayout.widget.CoordinatorLayout
        android:id="@+id/coordinator_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@null">


        <!--     顶部内容区域       -->
        <com.google.android.material.appbar.AppBarLayout
            android:id="@+id/appbar_layout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="@null"
            android:orientation="vertical"
            app:elevation="0dp">

            <FrameLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                app:layout_scrollFlags="scroll">

                <!--     顶部数据区域    -->
                <androidx.recyclerview.widget.RecyclerView
                    android:id="@+id/rv_header"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content" />

                <!--    跑马灯数据    -->
                <ViewFlipper
                    android:id="@+id/vf_flipper"
                    android:layout_width="200dp"
                    android:layout_height="wrap_content"
                    android:layout_marginStart="12dp"
                    android:layout_marginTop="80dp"
                    android:inAnimation="@anim/anim_in"
                    android:outAnimation="@anim/anim_out" />
            </FrameLayout>


        </com.google.android.material.appbar.AppBarLayout>

         <!--     商品详情数据和推荐商品 h5    -->
        <com.fingogroup.fingo.lib.widget.web.NestedWebView
                android:id="@+id/wb_web"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                app:layout_behavior="@string/appbar_scrolling_view_behavior" />

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```