---
title: android singleTask Bug
date: 2016-11-15 14:09:56
categories: Android

---
<Excerpt in index | 首页摘要> 
> android singleTask Bug
> <!-- more -->
> <The rest of contents | 余下全文> 



今天有几个用户反馈，给连载添加标签添加不上。

测试了一下，又是正常的。



用户机型为：

```
Sony,L50T.

OPPO,U707T

Samsung,GT-N7102
```

用腾讯优测找到这几个机型。

发现确实添加不上。

打印日志 然后下载

```
run onActivityResult  requestCode:654, resultCode:0 //这句话是我打印的，resultCode是0代表是cancel了。

```

而且意外在这句日志后面发现了日志:

```
org.json.JSONException: No value for results
11-15 13:57:41.231: W/System.err(2358): 	at org.json.JSONObject.get(JSONObject.java:355)
11-15 13:57:41.231: W/System.err(2358): 	at org.json.JSONObject.getJSONArray(JSONObject.java:549)
11-15 13:57:41.231: W/System.err(2358): 	at com.qingxiang.ui.activity.ShowTagActivity.response(ShowTagActivity.java:160)
```

看了一下，这个日志是获取请求服务器获取推荐标签的。

纳闷了，如果这个日志还在跑，应该就说明当前这个界面还没有退出啊。

难道是在Activity销毁的时候没有取消网络请求？（这个界面不是我写的，所以不知道有没有取消。）

然后仔细看看了看日志，看到了这句话：

```
11-15 13:57:41.041: W/ActivityManager(989): Activity is launching as a new task, so cancelling activity result.
```

然后去检查了Activity的注册清单，发现确实配置了singleTask。

看来在一些老的机型上，如果启动了一个singleTask的Activity是没办法利用onActivityResult来传递数据的。

仔细看日志，是个很好的习惯！！！