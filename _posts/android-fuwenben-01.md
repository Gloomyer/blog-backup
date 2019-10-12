---
title: Android TextView 富文本
date: 2016-6-28 19:11:44
categories: Android
tags:
- Android
- TextView
---
<Excerpt in index | 首页摘要> 
>Android TextView 富文本  
>
<!-- more -->
<The rest of contents | 余下全文>  
  
今天，项目做到评论模块，要在一个TextView中，让用户昵称变色，并且可响应点击事件  
  
TextView原生就是支持的，直接看代码：

Activity onCreate代码块  
```java
	Text tv = findViewById(R.id.item_tv);
	tv.setText(getClickableSpan());
	//这句话让高亮的用户名可以响应点击事件
	tv.setMovementMethod(LinkMovementMethod.getInstance());  
```
富文本格式化方法
```java
	private SpannableString getClickableSpan() {
	        View.OnClickListener l = new OnClickListener() {
	            @Override
	            public void onClick(View v) {
	                Toast.makeText(MainActivity.this, "你点击了用户昵称", 0).show();
            }
	
	        String parentNick = comment.getCommentParentNick();
	        SpannableString spannableInfo
	                = new SpannableString("王二: 评论内容");
	
	        //这里的0.2 是指用户昵称的开始位置和结束位置
	        spannableInfo.setSpan(new Clickable(l),
	                0, 2,Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
	        return spannableInfo;
	}
```
富文本点击类
```java
	class Clickable extends ClickableSpan implements View.OnClickListener {
        private View.OnClickListener mListener;

        public Clickable(View.OnClickListener l) {
            mListener = l;
        }

        @Override
        public void onClick(View v) {
            mListener.onClick(v);
            //设置背景为透明，如果不设置点击昵称背景会变色
            ((TextView) v).setHighlightColor(Color.argb(0, 0, 0, 0));
        }

        @Override
        public void updateDrawState(TextPaint ds) {
            ds.setColor(getResources().getColor(R.color.blue));    //设置昵称的变色
            ds.setUnderlineText(false);    //去除超链接的下划线
        }
    }
```