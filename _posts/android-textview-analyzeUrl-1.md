---
title: Android TextView分析Url高亮显示并相应点击事件
date: 2016-8-9 19:56:07
categories: Android
tags:
- Android
- TextView
---
<Excerpt in index | 首页摘要> 
> Android TextView分析Url高亮显示并相应点击事件  
>
<!-- more -->
<The rest of contents | 余下全文>  
  
项目中需要对TextView的超链接响应点击跳转.  
  
这个相对简单，因为android原生textview就是支持的  
  
这个属性叫做autoLink,可选有:all|web|email|phone等..  
  
我们只需要响应超链接 直接填写web即可.  
  
接下来你点击在TextView中的超链接，系统会自动调用打开浏览器,浏览你点击的网页.  
  
但是因为我们项目Web浏览是自己做的，没有使用浏览器.所以需要截获这个点击事件，并传递至我们自己的WebActivity。  
  
截获事件代码如下:
  
继承自TextView，然后粘如如下代码: 
  
然后使用的时候，使用setTextByUrl()替代setText()即可
  
```java
	public void setTextByUrl(String text) {
	    content.setText(text);
	    SetLinkClickIntercept(this);
	}
	
	private void SetLinkClickIntercept(TextView textView) {
	
	    textView.setMovementMethod(LinkMovementMethod.getInstance());
	    CharSequence text = textView.getText();
	    if (text instanceof Spannable) {
	        int end = text.length();
	        Spannable sp = (Spannable) textView.getText();
	        URLSpan[] urls = sp.getSpans(0, end, URLSpan.class);
	        if (urls.length == 0) {
	            return;
	        }
	
	        SpannableStringBuilder spannable = new SpannableStringBuilder(text);
	        // 只拦截 http:// 与 https://
	        LinkedList<String> myurls = new LinkedList<String>();
	        for (URLSpan uri : urls) {
	            String uriString = uri.getURL();
	            if (uriString.indexOf("http://") == 0 || uriString.indexOf("https://") == 0) {
	                myurls.add(uriString);
	            }
	        }
	        // 循环把链接发过去
	        for (URLSpan uri : urls) {
	            String uriString = uri.getURL();
	            if (uriString.indexOf("http://") == 0 || uriString.indexOf("https://") == 0) {
	                MyURLSpan myURLSpan = new MyURLSpan(uriString, myurls);
	                spannable.setSpan(myURLSpan, sp.getSpanStart(uri), sp.getSpanEnd(uri),
	                        Spannable.SPAN_INCLUSIVE_EXCLUSIVE);
	            }
	        }
	        textView.setText(spannable);
	    }
	}
	
	/**
	 * 处理TextView中的链接点击事件 
	 */
	private class MyURLSpan extends ClickableSpan {
	    private String mUrl; // 当前点击的实际链接
	    private LinkedList<String> mUrls; // 根据需求，一个TextView中存在多个link的话，
	    // 无论点击哪个都必须知道该TextView中的所有link，因此添加改变量
	
	    MyURLSpan(String url, LinkedList<String> urls) {
	        mUrl = url;
	        mUrls = urls;
	    }
	
	    @Override
	    public void updateDrawState(TextPaint ds) {
	        ds.setColor(ds.linkColor);
	        ds.setUnderlineText(false); // 是否去掉下划线
	        ds.setColor(getResources().getColor(R.color.green)); //设置超链接颜色
	
	    }
	
	    @Override
	    public void onClick(View widget) {
			//启动我们自己的Web浏览界面
	        WebActivity.startActivity(MyApp.getInstance(), mUrl);
	    }
	}
```