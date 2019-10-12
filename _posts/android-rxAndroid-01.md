---
title: Android-RxAndroid-响应编程-01
date: 2016-8-17 18:13:33
categories: Android
tags:
- Android
- RxAndroid
- 响应编程
---
<Excerpt in index | 首页摘要> 
> Android-RxAndroid-响应编程-01
>
<!-- more -->
<The rest of contents | 余下全文>  
  
####概述
RxAndroid应该是可以说是现在Android目前最火的技术之一了.  
它是依赖于RxJava的，可以说是RxAndroid是RxJava的一个Android适用版本.  
官方介绍:  
RxJava is a Java VM implementation of Reactive Extensions: a library for composing asynchronous and event-based programs by using observable sequences.  
(RxJava是一个基于Java VM实现扩展:库编写异步使用可观察序列和基于事件的程序,连猜带蒙不一定完全正确.)  

####摘要
那么问题来了，这玩意到底能够给我们带来哪些方面的便捷呢?  
Android一直让开发者头疼的事情就是它的异步调用问题了.  
原来的AnsycTask和Handler都是为了异步调用的方便而设计出来的..(从4.0开始，网络请求也开始需要异步调用)  
那么你有没有发现异步调用的复杂逻辑早都让人写炸了呢？  
  
假设，我们现在有这样一个简单的需求,从本地数据库中读取一大堆的数据文件.然后反序列化为基础的JavaBean，然后在主线程给相应的View设置上相应的数据.  
  
原先实现方式:  
```java
	public Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            if (msg.what == RESULT_OK) {
                UserBean userInfo = (UserBean) msg.obj;
                tv.setText(userInfo.getName());
                Glide.with(getActivity()).url(userInfo.getPic()).into(iv);
            }
        }
    };

	new Thread(new Runnable(){
		public void run(){
			//这里假设就是耗时操作 读取数据库并且转换为bean
			UserBean userInfo = getDataByDatabase();
			Message msg = mHandler.obtainMessage();
	        msg.obj = userInfo;
	        msg.what = RESULT_OK;
	        msg.sendToTarget();
		}
	}).start();  
```
  
若只是这么简单还好，但是往往我们要处理的逻辑远远不止这么一点，RxAndroid就比较完美的帮我们解决了这个问题，来看看RxAndroid怎么做的？  
```java
	rx.Observable.
            create(new Observable.OnSubscribe<UserBean>() {
                @Override
                public void call(Subscriber<? super UserBean> subscriber) {
                    UserBean userInfo = getDataByDatabase();
                    subscriber.onNext(drawable);
                    subscriber.onCompleted();
                }
            })
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(new Observer<UserBean>() {

                @Override
                public void onNext(UserBean userInfo) {
                    tv.setText(userInfo.getName());
                    Glide.with(getActivity()).url(userInfo.getPic()).into(iv);
                }

                @Override
                public void onCompleted() {
                    Log.i(TAG, "处理完成");
                }

                @Override
                public void onError(Throwable e) {
                    Log.i(TAG, "处理失败." + e.toString());
                }

            });  
```
代码量来说，后者随着逻辑的处理会越来越多，但是后者的魅力在于，一条线，无论如何复杂的异步线程调用.自始至终都是一条线的流程走下来，完全不会影响到你的代码逻辑。  
  
暂时就介绍这么多。下一篇介绍如何使用.