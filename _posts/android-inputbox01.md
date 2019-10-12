---
title: Andrid 弹出一个立于输入法上方的InputBox
date: 2016-6-29 17:04:07
categories: Android
tags:
- Android
---
<Excerpt in index | 首页摘要> 
>Andrid 弹出一个立于输入法上方的InputBox  
>
<!-- more -->
<The rest of contents | 余下全文>  
  
公司项目APP的首页，功能有添加评论。要在输入法弹出的时候在输入法正上方显示一个评论输入框  
  
先简单介绍下布局：
```java
	<activity>
		<具有添加评论功能的Fragment>
		<主界面Fragment切换Button>
	</activity>  
```
因为底部有这个切换Button,所以不能通过设置Activity的inputMode来实现，不然底部的切换Button也会卡在输入法上方  
  
简答说下我的思路，获取输入法的高度，然后将底部inputbox的paddingBottom设置为原始paddingBottom+输入法高度。  
  
通过查询资料发现，是没办法直接获取输入法高度。  
  
但是有一个小的作弊方式获取.  
  
View拥有getWindowVisibleDisplayFrame()可以获取显示的高度。  
  
那么就通过监听fragment的rootView的getViewTreeObserver().addOnGlobalLayoutListener()。  
  
获取getWindowVisibleDisplayFrame()用差值来判断输入的高度。然后设置View  
  
具体来看代码：
Fragment onViewCreated()方法中加入：
```java
	contentView.getViewTreeObserver().addOnGlobalLayoutListener(() -> {
            /**
             * 当评论输入框 弹出,以下代码是控制评论输入框移动至输入法上方
             */
            Rect rect = new Rect();
            contentView.getWindowVisibleDisplayFrame(rect);
            if (followLlInput.getVisibility() == View.VISIBLE) {
                if (mainBottomLayoutHeight <= 0)
                    mainBottomLayoutHeight = ((MainActivity) getActivity()).getBottomLayout().getHeight();

                if (screenHeight <= 0)
                    screenHeight = Utils.getScreenHeight();
                int bottom = screenHeight - rect.bottom;
                if (bottom > 0) {
                    bottom -= mainBottomLayoutHeight;
					//bottom 就是输入法的高度咯
                    followLlInput.setPadding(size10, size10, size10, bottom + size10);
                } else {
                    followLlInput.setPadding(size10, size10, size10, size10);
                }
            }
    });  
```
接下来就简单了，因为inputBox默认Visible是gone的  
  
在添加评论的onClickListener中加入如下代码：  
```java
	followLlInput.setVisibility(View.VISIBLE);
	//弹出输入法
    InputMethodManager imm =
            (InputMethodManager) getActivity().getSystemService(Context.INPUT_METHOD_SERVICE);
    imm.toggleSoftInput(0, InputMethodManager.HIDE_NOT_ALWAYS);
    followEtInput.requestFocus();  
```
这里有一点需要注意，让EditText获取选中焦点，用setSelect(true)是无效的,必须用requestFocus()才可以请求上焦点，不然还得用户点一下输入框，才能够正常输入评论  
  
接下来就更简单了，在动态显示的Recycler的onScrollListener中加入如下代码：  
```java
	followLlInput.setVisibility(View.GONE);
    /**
     * 关闭输入法
     */
    InputMethodManager inputMethodManager =
            (InputMethodManager) getContext().getSystemService(Context.INPUT_METHOD_SERVICE);
    inputMethodManager.hideSoftInputFromWindow(getActivity().getCurrentFocus().getWindowToken()
            , InputMethodManager.HIDE_NOT_ALWAYS);  
```
注意，上面的代码要套上if(dy != 0),因为输入法的显示/隐藏会触发RecyclerView的onScrollListener,但是不会移动距离  
  
这就实现了我们需要的功能了。