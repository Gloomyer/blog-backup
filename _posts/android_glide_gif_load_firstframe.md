---
title: Glide4.0加载GIF默认不播放 点击播放
date: 2018-7-26 16:15:59
categories: Android
tags: 
- Android
- Glide
---
<Excerpt in index | 首页摘要> 
>  加载Gif只显示第一帧，点击播放
> 
> <!-- more -->
> <The rest of contents | 余下全文> 

app一个业务流里面会出现多个GIF同屏幕的的情况

可能导致手机发烫比较严重。

产品那边给的需求变成了，只显示第一帧，点击播放

glide3.6的时候 在Glide.load之后加入asBitmap即可实现不播放。

4.0就不行了。

Glide加载url，检测到如果是GIF会解析成GIFDrawable，然后设置给ImageVIew，然后播放。

GIFDrawable可以控制播放次数和获取GIF的第一帧

那么我们可以这么做。

```
Glide.with(mFragment)
    .load(url)
    .into(new SimpleTarget<Drawable>() {
        @Override
        public void onResourceReady(Drawable resource, Transition<? super Drawable> transition) {
            if (resource instanceof GifDrawable) {
                if (isPlayGif) { //是否播放GIf的控制
                    iv.setImageDrawable(resource);
                    ((GifDrawable) resource).start(); //开始播放GIF
                } else {
					//不播放GIF，只展示GIF第一帧
                    iv.setImageBitmap(((GifDrawable) resource).getFirstFrame());
                }
            } else {
				//非GIF情况
                iv.setImageDrawable(resource);
            }
        }
    });
```

顺带一说，发现个别大GIF播放起来特别特别卡。

在加载的时候配上参数diskCacheStrategy(DiskCacheStrategy.RESOURCE); 即可解决

具体原因没找到。。。[Glide加载GIF卡解决方法](https://www.jianshu.com/p/5605456b9abb) 进去搜索缓存设置