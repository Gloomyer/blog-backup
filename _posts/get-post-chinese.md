---
title: GET 请求 提交中文
date: 2016-10-28 10:30:38
categories: Android
tags: 
- GET请求
- HTTP协议

---
<Excerpt in index | 首页摘要> 
> GET 请求 提交中文
> <!-- more -->
> <The rest of contents | 余下全文> 

##  问题描述  ##

这段时间，在更换网络框架。

新的选的是Volley，自己在做一个Request构建工具类。

项目中有一个模块是使用的GET请求，但是查询的value是中文。

结果好久没处理这块的内容了，忘记编码了，自然查询不到数据。



##  有意思的事情  ##

http://xxx.xxx.xxx.xxx/xxx/xx/xxx/xxxxxx?name=新星瞩目 

这是我们的接口，如果你放到postman/Chrome 中去，会发现都是正常的。

并不会乱码。

刚开始还挺疑惑，难道我记错了？Http协议里面 head里面的内容不是不能为中文嘛？

然后打开开发者工具，network分析，底部Query String Parameters.点击 view Source后发现。

Chrome是转码了的，只不过显示上，没有显示转码后的内容而已！

postman 有个生成请求代码的工具，点击生成，无论生成为什么，都会发现， 里面转换的代码也是经过了Url编码了的。



##  具体解决方案  ##

没什么难的，就是在get请求Build请求url，添加参数的时候，将有可能为中文内容url编码一下。



UTF-8 URL编码 代码:

```java
private static String Utf8URLencode(String text) {
  StringBuffer result = new StringBuffer();
  for (int i = 0; i < text.length(); i++) {
    char c = text.charAt(i);
    if (c >= 0 && c <= 255) {
      result.append(c);
    } else {
      byte[] b = new byte[0];
      try {
        b = Character.toString(c).getBytes("UTF-8");
      } catch (Exception ex) {
      }
      for (int j = 0; j < b.length; j++) {
        int k = b[j];
        if (k < 0) k += 256;
        result.append("%" + Integer.toHexString(k).toUpperCase());
      }
    }
  }
  return result.toString();
}
```

