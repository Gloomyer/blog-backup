---
title: Android  全文/收起 TextView02
date: 2016-9-19 10:17:44
categories: Android
tags: 
- Android
- 自定义View
---
<Excerpt in index | 首页摘要> 
>Android TextView全文/收起与SpannableString冲突导致TextView显示错乱的解决思路与解决方案
<!-- more -->
<The rest of contents | 余下全文> 
  
###  引言  ###
之前有一篇实现全文/收起功能的TextView:
[老文章](http://gloomyer.com/2016/06/22/android-fulltext01/)  
  
简单阐述一下老文章的实现思路.  
通过 控制TextView的maxLines属性来达到.  
  
另一种实现思路:  
通过给TextView 外面加一层容器，然后控制容器的高度来实现.  
  
两种方式我都实现了.具体代码就不贴了，反正都是有问题的.  
  
###  Bug描述   ###
那么出现什么问题了呢?  
  
如若只是普通的文本展示。还没有发现很大的问题.  
  
如果需要展示的文本,需求要用到SpannableString来高亮显示Url并且响应点击事件,亦或是使用了[TextView的autoLink属性然后截断事件来高亮Url并且响应点击事件](http://www.gloomyer.com/2016/08/09/android-textview-analyzeUrl-1/).  
  
Bug复现步骤:  
```shell
1:拥有一个超过预览行数并且包含url的文本.且url地址结束位置在5行之外.  
2:点击url.  
3:返回  
4:点击收起  
```
Bug,出现.看图:  
![](http://gloomyer.com/img/img/fulltextview_bug.png)  
  
错位了.不管上面两种哪种实现方式，最后都会出现这个问题.  
  
###  Bug思考  ###
问题出在哪里?  
  
简单滤过了一遍TextView的onDraw方法,发现没有问题.(就是从第一个字母绘制至最后一个字符).  
  
考虑到是点击url之后才出现的bug.  
  
怀疑是TextView 焦点移动至了url区域。才导致的.  
  
尝试给TextView增加:
```
android:focusable="false"
```
无果  
  
尝试在点击收起之后调用:
```
tv.clearFocus();
```
无效.  
  
也就是说，目前没有找到清除焦点的api.卡住.  
  
##  解决方案  ##
双TextView.  
  
既然焦点移动是因为变动了TextView的高度导致的.  
  
那么我们就不去变动TextView的高度.
  
利用两个TextView来实现一个显示收起的,一个显示全部的.  
  
简单说一下布局  
```
<线性布局>
    <帧布局>
        <收起TextView>
        <全文TextView>
    </帧布局>
    <全文/收起按钮 />
</线性布局>
```
然后看一下View初始化的代码:
```
View view = View.inflate(context, R.layout.layout_stretch_textview, this);
contentOverview = (TextView) view.findViewById(R.id.stretch_content_overview);
contentAll = (TextView) view.findViewById(R.id.stretch_content_all);
fullText = (TextView) view.findViewById(R.id.stretch_fulltext);
fullText.setOnClickListener(this);
contentAll.setMovementMethod(LinkMovementMethod.getInstance());
```
再看一下设置文本的代码:
```
setText(text);
this.isFullText = isFullText;
fullText.setText(isFullText ? "收起" : "全文");
//读取行数，用以判断是否显示全文按钮
contentAll.getViewTreeObserver().addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener() {
    @Override
    public boolean onPreDraw() {
        lineCount = contentAll.getLineCount();
        L.i(TAG, "lineCount" + lineCount);
        if (lineCount <= previewSize)
            fullText.setVisibility(View.GONE);
        else
            fullText.setVisibility(View.VISIBLE);
        setViewStatus(isFullText);
        contentAll.getViewTreeObserver().removeOnPreDrawListener(this);
        return true;
    }
});
```
setViewStatus方法代码:
```
private void setViewStatus(boolean isFullText) {
contentAll.setVisibility(isFullText ? View.VISIBLE : View.GONE);
contentOverview.setVisibility(isFullText ? View.GONE : View.VISIBLE);
}
```
setText(String text)方法就是[利用SpannableString高亮并且响应点击事件](http://www.gloomyer.com/2016/06/28/android-fuwenben-01/)的代码了,这里就不具体贴出来了.如果不知道怎么实现的，可以点击直接去看我的博客文章.