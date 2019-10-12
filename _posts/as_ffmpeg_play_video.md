---
title: 利用Ffmpeg解码视频绘制图像到UI
date: 2017-07-06 13:26:20
categories: Android
tags: 
- ndk
- ffmpeg
---
<Excerpt in index | 首页摘要> 
>  利用Ffmpeg解码视频绘制图像到UI
> <!-- more -->
> <The rest of contents | 余下全文> 

##  前提  ##
阅读之前可以先看一下我的

[自己编译ffmpeg在Android平台实现转码功能](http://gloomyer.com/2017/07/04/as_build_ffmepg_convert/)

里面讲解了如何编译ffmpeg的so包 本篇文章不在阐述.

##  需要的工具  ##
1->编译好的ffmpeg动态库(还有include文件)

2->libyuv(会讲如何编译/文章末尾源码文件中提供下载)

3->as

4->ndk

##  编译libyuv  ##
视屏解码出来是yuv的编码，我们需要转换为rgb才能够显示在UI上

首先创建一个libyuv的文件夹

```
cd ~
mkdir libyuv
cd libyuv
```

然后下载libyuv,他是利用git托管的。

```
git clone https://chromium.googlesource.com/libyuv/libyuv
mv libyuv jni //修改为jni目录 然后利用ndk编译
```

修改jni目录中的Android.mk

内容修改为如下：

```
# This is the Android makefile for libyuv for NDK.
LOCAL_PATH:= $(call my-dir)

include $(CLEAR_VARS)

LOCAL_CPP_EXTENSION := .cc

LOCAL_SRC_FILES := \
    source/compare.cc           \
    source/compare_common.cc    \
    source/compare_gcc.cc       \
    source/compare_neon.cc      \
    source/compare_neon64.cc    \
    source/convert.cc           \
    source/convert_argb.cc      \
    source/convert_from.cc      \
    source/convert_from_argb.cc \
    source/convert_to_argb.cc   \
    source/convert_to_i420.cc   \
    source/planar_functions.cc  \
    source/rotate.cc            \
    source/rotate_any.cc        \
    source/rotate_argb.cc       \
    source/rotate_common.cc     \
    source/rotate_dspr2.cc      \
    source/rotate_gcc.cc        \
    source/rotate_msa.cc        \
    source/rotate_neon.cc       \
    source/rotate_neon64.cc     \
    source/row_any.cc           \
    source/row_common.cc        \
    source/row_dspr2.cc         \
    source/row_gcc.cc           \
    source/row_msa.cc           \
    source/row_neon.cc          \
    source/row_neon64.cc        \
    source/scale.cc             \
    source/scale_any.cc         \
    source/scale_argb.cc        \
    source/scale_common.cc      \
    source/scale_dspr2.cc       \
    source/scale_gcc.cc         \
    source/scale_msa.cc         \
    source/scale_neon.cc        \
    source/scale_neon64.cc      \
    source/video_common.cc

#common_CFLAGS := -Wall -fexceptions
#ifneq ($(LIBYUV_DISABLE_JPEG), "yes")
#LOCAL_SRC_FILES += \
#    source/convert_jpeg.cc      \
#    source/mjpeg_decoder.cc     \
#    source/mjpeg_validate.cc
#common_CFLAGS += -DHAVE_JPEG
#LOCAL_SHARED_LIBRARIES := libjpeg
#endif

#LOCAL_CFLAGS += $(common_CFLAGS)
LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)/include
LOCAL_C_INCLUDES += $(LOCAL_PATH)/include
LOCAL_EXPORT_C_INCLUDE_DIRS := $(LOCAL_PATH)/include

LOCAL_MODULE := libyuv
LOCAL_MODULE_TAGS := optional

include $(BUILD_SHARED_LIBRARY)

#include $(CLEAR_VARS)

#LOCAL_WHOLE_STATIC_LIBRARIES := libyuv_static
#LOCAL_MODULE := libyuv
#ifneq ($(LIBYUV_DISABLE_JPEG), "yes")
#LOCAL_SHARED_LIBRARIES := libjpeg
#endif

#include $(BUILD_SHARED_LIBRARY)

#include $(CLEAR_VARS)
#LOCAL_STATIC_LIBRARIES := libyuv_static
#LOCAL_SHARED_LIBRARIES := libjpeg
#LOCAL_MODULE_TAGS := tests
#LOCAL_CPP_EXTENSION := .cc
#LOCAL_C_INCLUDES += $(LOCAL_PATH)/include
#LOCAL_SRC_FILES := \
#    unit_test/unit_test.cc        \
#    unit_test/basictypes_test.cc  \
#    unit_test/color_test.cc       \
#    unit_test/compare_test.cc     \
#    unit_test/convert_test.cc     \
#    unit_test/cpu_test.cc         \
#    unit_test/cpu_thread_test.cc  \
#    unit_test/math_test.cc        \
#    unit_test/planar_test.cc      \
#    unit_test/rotate_argb_test.cc \
#    unit_test/rotate_test.cc      \
#    unit_test/scale_argb_test.cc  \
#    unit_test/scale_test.cc       \
#    unit_test/video_common_test.cc

#LOCAL_MODULE := libyuv_unittest
##include $(BUILD_NATIVE_TEST)
```

在jni目录下创建Application.mk 内容如下:

```
APP_ABI	:= armeabi
```

然后回退到我们创建的libyuv目录下.

```
ndk-build //利用ndk编译
```

然后在生成的libs/armeabi目录下找到libyuv.so

还有jni目录下的include目录

这两部分就是我们所需要的内容

##  创建Android项目  ##

创建Android项目

在gradle.properties文件中加入

```
android.useDeprecatedNdk=true
```

打开app下的build.gradle

在defaultConfig节点下加入:

```
ndk {
    abiFilters "armeabi"
}
```

android节点下加入:

```
sourceSets.main {
    jni.srcDirs = []
    jniLibs.srcDirs = ['src/main/libs']
}
```

在app/src/main目录下创建jni目录

继续在jni目录下创建include目录

然后修改之前libyuv的include名字为libyuv

然后将libyuv拷贝至jni目录下的include文件夹中

将ffmpeg的include改名为ffmpeg继续放到jni目录下的include文件夹中

然后将ffmpeg/libyuv的so放在jni目录下

创建一个VideoUtils类 内容如下:

```
package com.gloomyer.ffmepgplay;

import android.view.Surface;

public class VideoUtils {
    public static native void render(String videoPath, Surface surface);
    static {
        System.loadLibrary("avutil-54");
        System.loadLibrary("swresample-1");
        System.loadLibrary("avcodec-56");
        System.loadLibrary("avformat-56");
        System.loadLibrary("swscale-3");
        System.loadLibrary("postproc-53");
        System.loadLibrary("avfilter-5");
        System.loadLibrary("avdevice-56");
        System.loadLibrary("myffmpeg");
    }
}
```

然后进入app/src/main/java 目录下利用javah生成头文件

```
javah -bootclasspath ~/Android/Sdk/platforms/android-?/android.jar com.gloomyer.ffmepgplay.VideoUtils
```

然后拷贝生成的头文件到jni目录下

然后创建myffmpeg.c文件

创建Android.mk文件　内容如下:

```

LOCAL_PATH := $(call my-dir)

#ffmpeg lib
include $(CLEAR_VARS)
LOCAL_MODULE := avcodec
LOCAL_SRC_FILES := libavcodec-56.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE := avdevice
LOCAL_SRC_FILES := libavdevice-56.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE := avfilter
LOCAL_SRC_FILES := libavfilter-5.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE := avformat
LOCAL_SRC_FILES := libavformat-56.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE := avutil
LOCAL_SRC_FILES := libavutil-54.so
include $(PREBUILT_SHARED_LIBRARY)


include $(CLEAR_VARS)
LOCAL_MODULE := postproc
LOCAL_SRC_FILES := libpostproc-53.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE := swresample
LOCAL_SRC_FILES := libswresample-1.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE := swscale
LOCAL_SRC_FILES := libswscale-3.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE := yuv
LOCAL_SRC_FILES := libyuv.so
include $(PREBUILT_SHARED_LIBRARY)

#myapp
include $(CLEAR_VARS)
LOCAL_MODULE := myffmpeg
LOCAL_SRC_FILES := myffmpeg.c
LOCAL_C_INCLUDES += $(LOCAL_PATH)/include/ffmpeg
LOCAL_C_INCLUDES += $(LOCAL_PATH)/include/libyuv
##-landroid参数 for native windows
LOCAL_LDLIBS := -llog -landroid
LOCAL_SHARED_LIBRARIES := avcodec avdevice avfilter avformat avutil postproc swresample swscale yuv
include $(BUILD_SHARED_LIBRARY)
```

Application.mk,内容如下:

```
APP_ABI := armeabi
APP_PLATFORM := android-9
```

最后的文件格式为:

![](http://gloomyer.com/img/img/as_ffmpeg_play_video_01.png)

打开myffmpeg.c

内容如下

```
#include "com_gloomyer_ffmepgplay_VideoUtils.h"
#include "include/ffmpeg/libavcodec/avcodec.h"
#include "include/ffmpeg/libavformat/avformat.h"
#include "include/ffmpeg/libswscale/swscale.h"
#include "include/libyuv/libyuv.h"
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <android/log.h>
#include <android/native_window.h>
#include <android/native_window_jni.h>

#define LOGI(FORMAT, ...) __android_log_print(ANDROID_LOG_INFO,"TAG",FORMAT,##__VA_ARGS__);
#define LOGE(FORMAT, ...) __android_log_print(ANDROID_LOG_ERROR,"TAG",FORMAT,##__VA_ARGS__);

JNIEXPORT void JNICALL Java_com_gloomyer_ffmepgplay_VideoUtils_render
        (JNIEnv *env, jclass jcls, jstring input_jstr, jobject suface_jobj) {
    const char *input_cstr = (*env)->GetStringUTFChars(env, input_jstr, NULL);
    LOGE("%s", "开始执行jni代码");
    av_register_all();//注册
    LOGE("%s", "注册组件成功");
    AVFormatContext *pFormatCtx = avformat_alloc_context();
    LOGE("%s", "注册变量");
    //2.打开输入视频文件
    if (avformat_open_input(&pFormatCtx, input_cstr, NULL, NULL) != 0) {
        LOGE("%s", "打开输入视频文件失败");
        return;
    } else {
        LOGE("%s", "打开视频文件成功!");
    }

    //3.获取视频信息
    if (avformat_find_stream_info(pFormatCtx, NULL) < 0) {
        LOGE("%s", "获取视频信息失败");
        return;
    } else {
        LOGE("%s", "获取视频信息成功！");
    }

    int index;
    for (int i = 0; i < pFormatCtx->nb_streams; i++) {
        if (pFormatCtx->streams[i]->codec->codec_type == AVMEDIA_TYPE_VIDEO) {
            index = i;
            break;
        }
    }

    //4.获取视频解码器
    AVCodecContext *pCodecCtx = pFormatCtx->streams[index]->codec;
    AVCodec *pCodec = avcodec_find_decoder(pCodecCtx->codec_id);

    if (pCodec == NULL) {
        LOGE("%s", "无法解码");
        return;
    } else {
        LOGE("%s", "可以正常解码");
    }

    //5.打开解码器
    if (avcodec_open2(pCodecCtx, pCodec, NULL) < 0) {
        LOGE("%s", "解码器无法打开");
        return;
    } else {
        LOGE("%s", "解码器打开成功!");
    }

    //编码数据
    AVPacket *pPacket = (AVPacket *) av_malloc(sizeof(AVPacket));

    //像素数据（解码数据）
    AVFrame *pYuvFrame = av_frame_alloc();
    AVFrame *pRgbFrame = av_frame_alloc();

    //native绘制
    //窗体
    ANativeWindow *pNativeWindow = ANativeWindow_fromSurface(env, suface_jobj);
    //绘制时的缓冲区
    ANativeWindow_Buffer outBuffer;
    //6.一阵一阵读取压缩的视频数据AVPacket
    int len, got_frame, framecount = 0;
    LOGE("%s", "开始一帧一帧解码");
    while (av_read_frame(pFormatCtx, pPacket) >= 0) {
        //解码AVPacket->AVFrame
        len = avcodec_decode_video2(pCodecCtx, pYuvFrame, &got_frame, pPacket);
        LOGE("%s len=%d got_frame=%d", "尝试解码", len, got_frame);
        if (got_frame) {
            LOGI("解码%d帧", framecount++);
            //设置缓冲区的属性（宽、高、像素格式）
            ANativeWindow_setBuffersGeometry(pNativeWindow,
                                             pCodecCtx->width,
                                             pCodecCtx->height,
                                             WINDOW_FORMAT_RGBA_8888);
            //lock
            ANativeWindow_lock(pNativeWindow, &outBuffer, NULL);

            //设置rgb_frame的属性（像素格式、宽高）和缓冲区
            //rgb_frame缓冲区与outBuffer.bits是同一块内存
            avpicture_fill((AVPicture *) pRgbFrame,
                           outBuffer.bits,
                           PIX_FMT_RGBA,
                           pCodecCtx->width,
                           pCodecCtx->height);


            I420ToARGB(pYuvFrame->data[0], pYuvFrame->linesize[0],
                       pYuvFrame->data[2], pYuvFrame->linesize[2],
                       pYuvFrame->data[1], pYuvFrame->linesize[1],
                       pRgbFrame->data[0], pRgbFrame->linesize[0],
                       pCodecCtx->width, pCodecCtx->height);

            ANativeWindow_unlockAndPost(pNativeWindow);

            usleep(1000 * 16);
        }
        av_free_packet(pPacket);
    }

    av_frame_free(&pYuvFrame);
    av_frame_free(&pRgbFrame);
    avcodec_close(pCodecCtx);
    avformat_free_context(pFormatJNIEXPORT void JNICALL Java_com_gloomyer_ffmepgplay_VideoUtils_render
        (JNIEnv *env, jclass jcls, jstring input_jstr, jobject suface_jobj) {
    const char *input_cstr = (*env)->GetStringUTFChars(env, input_jstr, NULL);
    LOGE("%s", "开始执行jni代码");
    av_register_all();//注册
    LOGE("%s", "注册组件成功");
    AVFormatContext *pFormatCtx = avformat_alloc_context();
    LOGE("%s", "注册变量");
    //2.打开输入视频文件
    if (avformat_open_input(&pFormatCtx, input_cstr, NULL, NULL) != 0) {
        LOGE("%s", "打开输入视频文件失败");
        return;
    } else {
        LOGE("%s", "打开视频文件成功!");
    }

    //3.获取视频信息
    if (avformat_find_stream_info(pFormatCtx, NULL) < 0) {
        LOGE("%s", "获取视频信息失败");
        return;
    } else {
        LOGE("%s", "获取视频信息成功！");
    }

    int index;
    for (int i = 0; i < pFormatCtx->nb_streams; i++) {
        if (pFormatCtx->streams[i]->codec->codec_type == AVMEDIA_TYPE_VIDEO) {
            index = i;
            break;
        }
    }

    //4.获取视频解码器
    AVCodecContext *pCodecCtx = pFormatCtx->streams[index]->codec;
    AVCodec *pCodec = avcodec_find_decoder(pCodecCtx->codec_id);

    if (pCodec == NULL) {
        LOGE("%s", "无法解码");
        return;
    } else {
        LOGE("%s", "可以正常解码");
    }

    //5.打开解码器
    if (avcodec_open2(pCodecCtx, pCodec, NULL) < 0) {
        LOGE("%s", "解码器无法打开");
        return;
    } else {
        LOGE("%s", "解码器打开成功!");
    }

    //编码数据
    AVPacket *pPacket = (AVPacket *) av_malloc(sizeof(AVPacket));

    //像素数据（解码数据）
    AVFrame *pYuvFrame = av_frame_alloc();
    AVFrame *pRgbFrame = av_frame_alloc();

    //native绘制
    //窗体
    ANativeWindow *pNativeWindow = ANativeWindow_fromSurface(env, suface_jobj);
    //绘制时的缓冲区
    ANativeWindow_Buffer outBuffer;
    //6.一阵一阵读取压缩的视频数据AVPacket
    int len, got_frame, framecount = 0;
    LOGE("%s", "开始一帧一帧解码");
    while (av_read_frame(pFormatCtx, pPacket) >= 0) {
        //解码AVPacket->AVFrame
        len = avcodec_decode_video2(pCodecCtx, pYuvFrame, &got_frame, pPacket);
        LOGE("%s len=%d got_frame=%d", "尝试解码", len, got_frame);
        if (got_frame) {
            LOGI("解码%d帧", framecount++);
            //设置缓冲区的属性（宽、高、像素格式）
            ANativeWindow_setBuffersGeometry(pNativeWindow,
                                             pCodecCtx->width,
                                             pCodecCtx->height,
                                             WINDOW_FORMAT_RGBA_8888);
            //lock
            ANativeWindow_lock(pNativeWindow, &outBuffer, NULL);

            //设置rgb_frame的属性（像素格式、宽高）和缓冲区
            //rgb_frame缓冲区与outBuffer.bits是同一块内存
            avpicture_fill((AVPicture *) pRgbFrame,
                           outBuffer.bits,
                           PIX_FMT_RGBA,
                           pCodecCtx->width,
                           pCodecCtx->height);


            I420ToARGB(pYuvFrame->data[0], pYuvFrame->linesize[0],
                       pYuvFrame->data[2], pYuvFrame->linesize[2],
                       pYuvFrame->data[1], pYuvFrame->linesize[1],
                       pRgbFrame->data[0], pRgbFrame->linesize[0],
                       pCodecCtx->width, pCodecCtx->height);

            ANativeWindow_unlockAndPost(pNativeWindow);

            usleep(1000 * 16);
        }
        av_free_packet(pPacket);
    }

    av_frame_free(&pYuvFrame);
    av_frame_free(&pRgbFrame);
    avcodec_close(pCodecCtx);
    avformat_free_context(pFormatCtx);
    ANativeWindow_release(pNativeWindow);
    (*env)->ReleaseStringUTFChars(env, input_jstr, input_cstr);JNIEXPORT void JNICALL Java_com_gloomyer_ffmepgplay_VideoUtils_render
        (JNIEnv *env, jclass jcls, jstring input_jstr, jobject suface_jobj) {
    const char *input_cstr = (*env)->GetStringUTFChars(env, input_jstr, NULL);
    LOGE("%s", "开始执行jni代码");
    av_register_all();//注册
    LOGE("%s", "注册组件成功");
    AVFormatContext *pFormatCtx = avformat_alloc_context();
    LOGE("%s", "注册变量");
    //2.打开输入视频文件
    if (avformat_open_input(&pFormatCtx, input_cstr, NULL, NULL) != 0) {
        LOGE("%s", "打开输入视频文件失败");
        return;
    } else {
        LOGE("%s", "打开视频文件成功!");
    }

    //3.获取视频信息
    if (avformat_find_stream_info(pFormatCtx, NULL) < 0) {
        LOGE("%s", "获取视频信息失败");
        return;
    } else {
        LOGE("%s", "获取视频信息成功！");
    }

    int index;
    for (int i = 0; i < pFormatCtx->nb_streams; i++) {
        if (pFormatCtx->streams[i]->codec->codec_type == AVMEDIA_TYPE_VIDEO) {
            index = i;
            break;
        }
    }

    //4.获取视频解码器
    AVCodecContext *pCodecCtx = pFormatCtx->streams[index]->codec;
    AVCodec *pCodec = avcodec_find_decoder(pCodecCtx->codec_id);

    if (pCodec == NULL) {
        LOGE("%s", "无法解码");
        return;
    } else {
        LOGE("%s", "可以正常解码");
    }

    //5.打开解码器
    if (avcodec_open2(pCodecCtx, pCodec, NULL) < 0) {
        LOGE("%s", "解码器无法打开");
        return;
    } else {
        LOGE("%s", "解码器打开成功!");
    }

    //编码数据
    AVPacket *pPacket = (AVPacket *) av_malloc(sizeof(AVPacket));

    //像素数据（解码数据）
    AVFrame *pYuvFrame = av_frame_alloc();
    AVFrame *pRgbFrame = av_frame_alloc();

    //native绘制
    //窗体
    ANativeWindow *pNativeWindow = ANativeWindow_fromSurface(env, suface_jobj);
    //绘制时的缓冲区
    ANativeWindow_Buffer outBuffer;
    //6.一阵一阵读取压缩的视频数据AVPacket
    int len, got_frame, framecount = 0;
    LOGE("%s", "开始一帧一帧解码");
    while (av_read_frame(pFormatCtx, pPacket) >= 0) {
        //解码AVPacket->AVFrame
        len = avcodec_decode_video2(pCodecCtx, pYuvFrame, &got_frame, pPacket);
        LOGE("%s len=%d got_frame=%d", "尝试解码", len, got_frame);
        if (got_frame) {
            LOGI("解码%d帧", framecount++);
            //设置缓冲区的属性（宽、高、像素格式）
            ANativeWindow_setBuffersGeometry(pNativeWindow,
                                             pCodecCtx->width,
                                             pCodecCtx->height,
                                             WINDOW_FORMAT_RGBA_8888);
            //lock
            ANativeWindow_lock(pNativeWindow, &outBuffer, NULL);

            //设置rgb_frame的属性（像素格式、宽高）和缓冲区
            //rgb_frame缓冲区与outBuffer.bits是同一块内存
            avpicture_fill((AVPicture *) pRgbFrame,
                           outBuffer.bits,
                           PIX_FMT_RGBA,
                           pCodecCtx->width,
                           pCodecCtx->height);


            I420ToARGB(pYuvFrame->data[0], pYuvFrame->linesize[0],
                       pYuvFrame->data[2], pYuvFrame->linesize[2],
                       pYuvFrame->data[1], pYuvFrame->linesize[1],
                       pRgbFrame->data[0], pRgbFrame->linesize[0],
                       pCodecCtx->width, pCodecCtx->height);

            ANativeWindow_unlockAndPost(pNativeWindow);

            usleep(1000 * 16);
        }
        av_free_packet(pPacket);
    }

    av_frame_free(&pYuvFrame);
    av_frame_free(&pRgbFrame);
    avcodec_close(pCodecCtx);
    avformat_free_context(pFormatCtx);
    ANativeWindow_release(pNativeWindow);
    (*env)->ReleaseStringUTFChars(env, input_jstr, input_cstr);
}
}Ctx);
    ANativeWindow_release(pNativeWindow);
    (*env)->ReleaseStringUTFChars(env, input_jstr, input_cstr);
}
```

然后进入app/src/main/目录下输入ndk-build生成so库

回到java方面,打开mainactivity的布局:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:orientation="vertical"
    android:layout_height="match_parent">

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:onClick="play"
        android:text="播放视频" />


    <SurfaceView
        android:id="@+id/mSurfaceView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</LinearLayout>
```

看一看MainActivity的代码:

```
package com.gloomyer.ffmepgplay;

import android.graphics.PixelFormat;
import android.os.Environment;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.SurfaceHolder;
import android.view.SurfaceView;
import android.view.View;

import java.io.File;

public class MainActivity extends AppCompatActivity {

    SurfaceView mSurfaceView;
    private SurfaceHolder mHolder;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mSurfaceView = (SurfaceView) findViewById(R.id.mSurfaceView);
        init();
    }

    private void init() {
        mHolder = mSurfaceView.getHolder();
        mHolder.setFormat(PixelFormat.RGBA_8888);
    }

    public void play(View v) {
        final String input = new File(Environment.getExternalStorageDirectory(), "input.mp4").getAbsolutePath();
        new Thread() {
            @Override
            public void run() {
                VideoUtils.render(input, mHolder.getSurface());
            }
        }.start();
    }
}
```

下载一个mp4上传到手机sd卡下,名字叫input.mp4

然后把项目跑起来，点击播放视频。就发现可以显示视频了(无声音，，我们没有做声音的解码播放，后续继续...)

##  源码打包下载  ##

[源码打包下载](http://gloomyer.com/upload/ ffmpeg_play_video.zip)