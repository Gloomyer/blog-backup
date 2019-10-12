---
title: Android 识别图片亮度
date: 2019-06-04 17:31:32
categories: Android
tags:
- Android
- JNI
---
<Excerpt in index | 首页摘要>
> Android 识别图片亮度
>
<!-- more -->
<The rest of contents | 余下全文>  

## 概要

有一个界面是文字盖在图片上面，

设计希望能够更具图片的亮度来让文字自动设置为白色或者黑色。

所以就需要识别图片的light值。

需要依赖jni开发技术

## 原理

java层拿到bitmap，然后丢给c++处理。

bitmap的格式通常都为rgba8888

我们将rgba8888转换为yuv，拿到Y值 y就是亮度，然后求平均值即可得到图片整体的亮度

第一版目前是这样，但是这样是有一定误差的，比如一张顶部1/3纯白 底部2/3纯黑的图片，会被认为是暗得，但是如果你的文字是盖在上部分，那么误差就会出现。

具体可以修改不求整体的平均值，求图片的局部亮度平均值更为精准。

图片的识别是一件耗时操作（虽然也很快），但是还是推荐在创建的时候去识别，服务端记录，展示的时候直接根据字段计算即可。

## 代码实现

AS创建JNI项目，

移动CMakeLists至app目录下

CMakeLists内容:

```cmake
cmake_minimum_required(VERSION 3.4.1)

add_library(libimg SHARED src/main/cpp/native-lib.cpp)

find_library(log-lib log)

target_link_libraries(libimg jnigraphics ${log-lib})
```

build.gradle修改

```groovy
externalNativeBuild {
  cmake {
    path "CMakeLists.txt"
		version "3.10.2"
  }
}
```

MainActivity代码:

```java
package com.gloomyer.lightdemo;

import android.Manifest;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.os.Environment;
import android.support.v4.app.ActivityCompat;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.ImageView;

import java.io.File;

public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    static {
        System.loadLibrary("libimg");
    }

    private ImageView iv;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.READ_EXTERNAL_STORAGE}, 123);
        iv = findViewById(R.id.iv);
    }


    public native int getLight(Bitmap bitmap);

    public boolean isLight(Bitmap bitmap) {
        return getLight(bitmap) > 128;
    }

    public void load(View view) {
//        Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.demo);
        Bitmap bitmap = BitmapFactory.decodeFile(new File(Environment.getExternalStorageDirectory(), "black.jpeg").getAbsolutePath());
        if (bitmap == null) {
            Log.e(TAG, "bitmap  == null");
            return;
        }
        Log.e(TAG, "is light:" + isLight(bitmap));
        iv.setImageBitmap(bitmap);
    }
}

```

native-lib.cpp代码

```c++
#include <jni.h>
#include <string>
#include <android/log.h>
#include <android/bitmap.h>

#define LOGE(FORMAT, ...) __android_log_print(ANDROID_LOG_ERROR,"MainActivity",FORMAT,##__VA_ARGS__);

extern "C"
JNIEXPORT jint JNICALL
Java_com_gloomyer_lightdemo_MainActivity_getLight(JNIEnv *env, jobject instance, jobject bitmap) {
    AndroidBitmapInfo info;
    int ret = 0;
    if ((ret = AndroidBitmap_getInfo(env, bitmap, &info)) >= 0) {
        uint32_t width = info.height;
        uint32_t height = info.width;
        LOGE("width:%d, height:%d, format:%d", info.width, info.height, info.format);
        void *addr = new uint32_t[width * height];
        long whereToGet = 0, bright = 0;
        int r, g, b;
        if (AndroidBitmap_lockPixels(env, bitmap, &addr) >= 0) {
            for (int i = 0; i < width; i++) {
                for (int j = 0; j < height; j++) {
                    uint32_t pixel = ((uint32_t *) addr)[whereToGet++];
                    r = (pixel | 0xff00ffff) >> 16 & 0x00ff;
                    g = (pixel | 0xffff00ff) >> 8 & 0x0000ff;
                    b = (pixel | 0xffffff00) & 0x0000ff;
                    bright = (int) (bright + 0.299 * r + 0.587 * g + 0.114 * b);
                }
            }
            AndroidBitmap_unlockPixels(env, bitmap);
        }
        LOGE("ret:%d", ret);
        return bright / (whereToGet - 1);
    }
    return -1;
}

```

## GitHub下载

[**android_distinguish_image_light**](https://github.com/Gloomyer/android_distinguish_image_light)

