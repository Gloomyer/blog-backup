---
title: Android获取TextView高度
date: 2016-9-12 09:42:33
categories: Android
tags:
- Android
- TextView
- getHeight
---
<Excerpt in index | 首页摘要> 
> Android获取TextView高度
>
<!-- more -->
<The rest of contents | 余下全文>  
  
昨天碰到了要界面初始化获取TextView高度的问题.
  
总的来说，目前一共有三种方式获取:  
  
##第一种:
```java
	tv.getViewTreeObserver().addOnGlobalLayoutListener(() -> {
		int height = tv.getHeight();
	});
```
这种是最准确的,但是同样有一个问题，如果你在初始化布局需要这个参数来做些什么的时候,就会出现卡顿现象.  
  
##第二种:
```java
	tv.getViewTreeObserver().addOnPreDrawListener(() -> {
		int height = tv.getHeight();
		return;
	});
```
这个是比较折中的办法..  
  
##第三种:
总有些情况是你意想不到的，我昨天就碰到的是这个情况.  
  
获取高度必须在Act的onCreate方法中获取,并且不能通过回调,因为回调总是有一点点的延迟.  
  
那就出现了这个扯淡的第三种(<font color='red'>如果非必要，强烈不建议该方法,我是被逼的了.</font>)  
  
那就是自己计算!  
  
```java
//获取TextView宽度
int tvWidth = Utils.getScreenWidth() - DensityUtils.dp2px(this, 20);
//获取一个文字的高宽
int textSize = DensityUtils.sp2px(this, 14);
int lineLength = tvWidth / textSize;// 获取一行能有多少个文字
int line = text.length() / lineLength; //获取有多少行
if(text.length() % lineLength > 0)
	line++;//如果不够除，还要加一行.  
//注意还要给这个textView的XML代码加上 android:lineSpacingExtra="2dp"
//因为我们不知道它默认的行间距是多少
line += stringCount(tagDesc, "\n");//如果存在换行符，也要考虑进去
//发现每一行实际上TextView的高度一般比字体大小会多1,比如这里14sp大小 高度就为15左右 可能会有误差.
int height = (line * DensityUtils.sp2px(this, 15))
			+ (DensityUtils.dp2px(this, 2) * line) //行间距
			+ (paddingValue); //TextView如果有padding 要加上，否则这里为0
```
统计字符串出现次数
```java
	public int stringCount(String str,String substr){
	int count = 0;
	int str_len = str.length();
	int substr_len = substr.length();
	if (!str.contains(substr))
	    return count;
	for (int i = 0; i < str_len - substr_len; i++) {
	    if (substr.equals(str.substring(i, i + substr_len))) {
	        count++;
	    }
	}
	return count;
}
```
这里这些是不完善的,比如标点符号全部以中文的方式计算的，所以后面还需要优化!