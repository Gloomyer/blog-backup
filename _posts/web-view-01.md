---
title: WebView-修改加载内容
date: 2017-01-04 19:21:27
categories: Android
tags:
- WebView
---
<Excerpt in index | 首页摘要> 
> WebView-修改加载内容
>
> <!-- more -->
> <The rest of contents | 余下全文>  



####  概要  ####

朋友电商项目，碰到一个问题.

商品详情是图文介绍.

需要加载的URL的html内容格式如下:

```html
"<p><img src=\"http://static.b2b2cv2.javamall.com.cn/attachment/ueditor/2016/5/26/15//49349658.jpg\" style=\"\" title=\"show.jpg\"/></p><p><img src=\"http://static.b2b2cv2.javamall.com.cn/attachment/ueditor/2016/5/26/15//49341729.jpg\" style=\"\" title=\"show.jpg\"/></p><p><img src=\"http://static.b2b2cv2.javamall.com.cn/attachment/ueditor/2016/5/26/15//49341234.jpg\" style=\"\" title=\"show.jpg\"/></p><p><img src=\"http://static.b2b2cv2.javamall.com.cn/attachment/ueditor/2016/5/26/15//49344408.jpg\" style=\"\" title=\"show.jpg\"/></p><p><br/></p>"
```

出现了个坑.

图片文字不居中显示了.



####  解决方案  ####



我的意思是，这是web的锅，你应该让web改一下格式吧？

他：“我们没有后台，这套后台是买的。”；

我猝，享年21.



好了不开玩笑，既然没办法让后台改。那我们来想想办法。

不是完整的html，不知道能否利用js去控制。

反正只是一个展示我就利用笨一点的办法吧。

WebView身上有这么一个方法.



```java
/*
简单描述下,
data:需要webView加载的数据(即html文本)
mimeType:要加载的类型(一般恒定text/html;)
encoding:编码格式，一般utf-8
*/
WebView.loadData(String data, String mimeType, String encoding);
```



我的想法就是在要加载的html文本外部套一个div标签,然后加上align属性

先利用项目的网络框架读取到要加载URL的html文本信息

然后修改html内容，利用这个方法直接去加载html



具体代码实现

```java
VU.get("http://xxxx/xxx/xxx/xxxx/xx")
  .execute(new StringCallback() {
    @Override
    public void onError(Call call, Exception e) {
		
    }

    @Override
    public void onResponse(String html) {
       //修改html内容
       html = "<div align=\"center\">" + html + "</div>";
       mWebView.loadData(html, "text/html", "utf-8");
    }
  });
```

好了。我懒。就这样吧。



说下缺陷

html在webView中应该是解释渲染

比如很长的html获取多少解析多少.(常见浏览器是，WebView不是很确定。没有深探过。)

如果改成现在这样，那就是必须等待html全部获取完毕才开始解析。速度会慢一些。

还好他现在的需求一般内容如上就几行文本。问题不大，就这么干吧。



最好(推荐)的方式：

利用js去控制。