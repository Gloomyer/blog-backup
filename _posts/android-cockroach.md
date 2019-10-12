---
title: Android Cockroach 原理实现分析
date: 2017-02-28 20:21:54
categories: Android
tags:
- Android
- Cockroach
- 永不崩溃的app
---
<Excerpt in index | 首页摘要> 
> Android Cockroach 原理实现分析
>
> <!-- more -->
> <The rest of contents | 余下全文>  



### 概述 ###

今天，朋友给介绍了个框架.

[Cockroach](https://github.com/android-notes/Cockroach)

它的介绍:

​        打不死的小强,永不crash的Android

通过分析原理，发现真J8屌，果然是大佬。



####  参考文档  ####

[官方原理分析](https://github.com/android-notes/Cockroach/blob/master/%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90.md)



####  需要知识  ####

需要了解Looper的基本原理，和Android App基本执行流程



####  实现原理  ####

首先，我们要知道一件事情。

Android应用本身就是一个java应用。

那么一个Java应用就必定需要一个main方法来作为入口。

那么，我们写的Android应用。main方法，在哪里？



ActivityThread 答案在这里

```java
public static void main(String[] args) {

        Looper.prepareMainLooper();//创建一个阻塞主线程的消息队列

        ActivityThread thread = new ActivityThread();
        thread.attach(false);//一些初始化操作

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```

上面的代码是删除了一部分的无关代码.



通过上述代码，我们可以得到这样一个信息:

我们所写的所有的执行代码，都是由ActivityThread 发送给了MainThread的Looper。

最后由主线程的Looper来执行了.

具体执行的代码在Looper.loop();这个方法的死循环中



那么，问题出现了。

当我们执行的代码碰到了crash。

那么当异常抛出后就会导致Looper.loop();这个死循环的代码退出死循环

当这个方法意外中断之后。

我们的app也就退出了！

如何让这里不异常退出就等于可以让我们的app不崩溃退出!



理想很丰满，现实很骨感。

你不可能改写android的代码，所以这里肯定会崩溃。

但是！ 就算崩溃了！我们是否可以重启它呢？



答案是可以的！

尝试创建一个空界面的app

然后粘如如下代码(感谢波波提供):

```java
final Handler handler = new Handler() {
            int i;

            @Override
            public void handleMessage(Message msg) {
                switch (msg.what) {
                    case -1:
                        throw new RuntimeException("magic");   //退出ActivityThread中的Looper.loop()
                    case 0:
                        Log.e("handleMessage()", "" + msg);
                        sendEmptyMessageDelayed(0, 1000);
                        if (++i % 5 == 0)
                            throw new RuntimeException("Hey,你猜会不会崩");
                        break;
                    default:
                        break;
                }
            }
        };

        handler.sendEmptyMessage(0);

        handler.post(new Runnable() {
            @Override
            public void run() {
                handler.sendEmptyMessage(-1);

                //抛异常后Looper.loop()会退出,所以要死循环重启
                for (; ; ) {
                    try {
                        Log.e("Restart", "Looper.loop()");
                        Looper.loop();
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }
        });
```



首先，通过主动抛一个异常，来中断系统当前正在执行的的loop();

然后，我们通过一个死循环来自己调用loop();

并且将他try起来！

然后放在一个死循环之间。

当再发生崩溃，会发生什么呢？



模拟一下:

程序运行->我们主动引发崩溃->启用我们自己的loop()->又发生崩溃->我们loop崩溃->但是我们try起来了->我们执行的loop()退出->然后执行循环->然后继续执行loop()



这个框架的大哥真心屌，小弟佩服.