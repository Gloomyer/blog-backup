---
title: Android高斯模糊
date: 2017-11-05 20:59:00
categories: Android
tags: 
- 高斯模糊
---
<Excerpt in index | 首页摘要> 
> Android高斯模糊
> <!-- more -->
> <The rest of contents | 余下全文> 

##  引言  ##

Android对高斯模糊支持的其实并不是很完美。

但是也是有办法使用的.(扩展库中)

##  如何使用  ##
首先在你的app的build中加入高斯模糊的支持

build.gradle
```
defaultConfig {

        //高斯模糊
        renderscriptTargetApi 18
        renderscriptSupportModeEnabled true

}
```

然后在代码中继续。

高斯模糊的库只支持对图片的高斯模糊

我的需求是对view怎么办呢？

其实也很容易，我们可以拿到view的缓存，也就是一张图片了然后在原有view上加一层imageView 然后高斯处理 最后设置回去

具体看代码

```
 mActivity.btn2.setDrawingCacheEnabled(true);
            Bitmap inputBitmap = Bitmap.createBitmap(mActivity.btn2.getDrawingCache());
            mActivity.btn2.setDrawingCacheEnabled(false);
            Bitmap outputBitmap = Bitmap.createBitmap(inputBitmap);
            RenderScript rs = RenderScript.create(mActivity);
            ScriptIntrinsicBlur blurScript = ScriptIntrinsicBlur.create(rs, Element.U8_4(rs));
            Allocation tmpIn = Allocation.createFromBitmap(rs, inputBitmap);
            Allocation tmpOut = Allocation.createFromBitmap(rs, outputBitmap);
            // 设置渲染的模糊程度, 25f是最大模糊度
            blurScript.setRadius(20f);
            // 设置blurScript对象的输入内存
            blurScript.setInput(tmpIn);
            // 将输出数据保存到输出内存中
            blurScript.forEach(tmpOut);

            // 将数据填充到Allocation中
            tmpOut.copyTo(outputBitmap);

            mActivity.testimage.setImageBitmap(outputBitmap);
```

这样就ok了!
