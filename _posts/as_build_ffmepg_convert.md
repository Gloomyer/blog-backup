---
title: 自己编译ffmpeg在Android平台实现转码功能
date: 2017-07-04 15:12:34
categories: Android
tags: 
- ndk
- ffmpeg
---
<Excerpt in index | 首页摘要> 
> 自己编译ffmpeg在Android平台实现转码功能
> <!-- more -->
> <The rest of contents | 余下全文> 

##  前提  ##
ffmpeg的鼎鼎大名应该算是无人不知无人不晓了,我就不介绍了.

文章底部提供源码下载
##  需要的东西  ##
1->ubuntu电脑一台(可以是服务器,可以是虚拟机.)(我的版本是16.04 64bit)

2->linux版本的ndk

3->[ffmpeg-2.6.9源码一份(点击这里从官网直接下载)](http://ffmpeg.org/releases/ffmpeg-2.6.9.tar.gz)

4->androidstudio

##  编译ffmpeg  ##
首先下载ffmpeg至你的服务器(我的pc装成了ubuntu).

然后下载linux版本的ndk(如果实在找不到可以[联系我](http://gloomyer.com/about/))

首先解压ndk完成之后.

修改下ndk目录的权限和配置一下环境变量

```
sudo chmod 777 -R /home/gloomy/Android/Sdk/ndk-bundle //(我是非root用户所以加sudo 如果你是root用户登录不需要加sudo )
vim ~/.profile //如果提示你没有vim，请利用 sudo apt-get install vim (如果是root用户登录不需要加sudo)  
```

在文件的底部加入

```
export NDK_HOME=/home/gloomy/Android/Sdk/ndk-bundle //这个是你的ndk安装目录
28 export PATH=$PATH:$NDK_HOME
```

然后解压下载好的ffmpeg2.6.9

```
tar -zxvf ffmpeg-2.6.9.tar.gz
mv ffmpeg-2.6.9 ~/ffmpeg
cd ~/ffmpeg
```

然后创建一个build_android.sh.内容如下：

```
vim build_android.sh
```

文件内容如下:

重点说一下 NDK=后面跟的是ndk的目录

TOOLCHAIN后面的arm-linux-androideabi-4.9这个目录需要你去你的ndk目录里面对照一下你自己的 看看到底是4.9还是多少。我这里是4.9 如果你下的老版本可能是4.8/4.7

```
#!/bin/bash
make clean
export NDK=/home/gloomy/Android/Sdk/ndk-bundle 
export SYSROOT=$NDK/platforms/android-9/arch-arm/
export TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64
export CPU=arm
export PREFIX=$(pwd)/android/$CPU
export ADDI_CFLAGS="-marm"

sudo ./configure --target-os=linux \
--prefix=$PREFIX --arch=arm \
--disable-doc \
--enable-shared \
--disable-static \
--disable-yasm \
--disable-symver \
--enable-gpl \
--disable-ffmpeg \
--disable-ffplay \
--disable-ffprobe \
--disable-ffserver \
--disable-doc \
--disable-symver \
--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
--enable-cross-compile \
--sysroot=$SYSROOT \
--extra-cflags="-Os -fpic $ADDI_CFLAGS" \
--extra-ldflags="$ADDI_LDFLAGS" \
$ADDITIONAL_CONFIGURE_FLAG
make clean
make
make install

```

然后修改configure文件的一个地方

```
vim configure
//按esc进入命令模式 输入
:to 2779
//从2279到2782 每行前面加# //#表示注释
//然后在后面加上

SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
SLIB_INSTALL_LINKS='$(SLIBNAME)'
//输入命令模式 输入wq //保存退出
```

然后开始执行编译

```
chmod 777 build_android.sh
sudo ./build_android.sh //如果你是root用户不需要加sudo 后面不会继续说了 估计跑好一阵 看电脑配置 我是i5 跑了7分钟左右
```

然后在~/ffmpeg/下会生成android目录 ffmpeg的编译我们就已经完成了

这整个目录就是我们所需要的了，你可以复制到你的开发电脑上了.

##  android转码的实现  ##
我们就来实现一个mp4转换为yuv视频的小demo吧.

利用as创建一个项目(不要选中 include c++ support)

然后在app/src/main目录下创建 jni和libs目录

打开gradle.properties文件

加入

```
android.useDeprecatedNdk=true
```

然后app下的build.gradle

在defaultConfig节点下加入

```
ndk {
    abiFilters "armeabi"
}
```

在android的节点下加入：

```
sourceSets.main {
    jni.srcDirs = []
    jniLibs.srcDirs = ['src/main/libs']
}
```

创建VideoUtils文件

内容如下:

```
package com.gloomyer.ffmpegplay;

/**
 * Created by gloomy on 17-7-4.
 */

public class VideoUtils {
    public static native void decode(String input, String output);

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

MainAct代码:

```
package com.gloomyer.ffmpegplay;

import android.os.Environment;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;

import java.io.File;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    public void convert(View v) {
        String input = new File(Environment.getExternalStorageDirectory(),"input.mp4").getAbsolutePath();
        String output = new File(Environment.getExternalStorageDirectory(),"output_1280x720_yuv420p.yuv").getAbsolutePath();
        VideoUtils.decode(input, output);
    }
}
```

main布局就不贴了，就是一个按钮 onClick调用convert

接下来就是重点了.

将之前编译ffmpeg生产的Android目录下的include目录拷贝至jni目录

lib中的 libavcodec-56.so liavdevice-56.so libavfilter-5.so libavformat-56.so libavutil-54.so libpostproc-53.so libswresample-1.so libswscale-3.so

8个so拷贝只jni目录下

创建Android.mk内容如下:

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

#myapp
include $(CLEAR_VARS)
LOCAL_MODULE := myffmpeg
LOCAL_SRC_FILES := ffmpeg_player.c
LOCAL_C_INCLUDES += $(LOCAL_PATH)/include
LOCAL_LDLIBS := -llog
LOCAL_SHARED_LIBRARIES := avcodec avdevice avfilter avformat avutil postproc swresample swscale
include $(BUILD_SHARED_LIBRARY)
```

创建Application.mk,内容如下:

```
APP_ABI := armeabi
APP_PLATFORM := android-9
```

打开terminal，进入app/src/main/java目录

输入:

```
javah com.gloomyer.ffmpegplay.VideoUtils //你的Videoutils的带包名全名 生产.h文件
```

刷新下目录将java下的生产的.h拷贝至jni目录

在jni目录下创建ffmpeg_play.c文件.

转码的实现(ffmpeg_play.c源码):

```
//
// Created by gloomy on 17-7-4.
//
#include <android/log.h>
#include "com_gloomyer_ffmpegplay_VideoUtils.h"
#include "include/libavcodec/avcodec.h"
#include "include/libavformat/avformat.h"
#include "include/libswscale/swscale.h"

#define LOGI(FORMAT, ...) __android_log_print(ANDROID_LOG_INFO,"jason",FORMAT,##__VA_ARGS__);
#define LOGE(FORMAT, ...) __android_log_print(ANDROID_LOG_ERROR,"jason",FORMAT,##__VA_ARGS__);

JNIEXPORT void JNICALL Java_com_gloomyer_ffmpegplay_VideoUtils_decode
        (JNIEnv *env, jclass jcls, jstring input_jstr, jstring output_jstr) {

    //需要转码的视频文件(输入的视频文件)
    const char *input_cstr = (*env)->GetStringUTFChars(env, input_jstr, NULL);
    const char *output_cstr = (*env)->GetStringUTFChars(env, output_jstr, NULL);

    //1.注册所有组件
    av_register_all();

    //封装格式上下文，统领全局的结构体，保存了视频文件封装格式的相关信息
    AVFormatContext *pFormatCtx = avformat_alloc_context();

    //2.打开输入视频文件
    if (avformat_open_input(&pFormatCtx, input_cstr, NULL, NULL) != 0) {
        LOGE("%s", "can't open input file");
        return;
    }

    //3.获取视频文件信息
    if (avformat_find_stream_info(pFormatCtx, NULL) < 0) {
        LOGE("%s", "can't get input video info");
        return;
    }

    //获取视频流的索引位置
    //遍历所有类型的流（音频流、视频流、字幕流），找到视频流
    int v_stream_idx = -1;
    int i = 0;
    //number of streams
    for (; i < pFormatCtx->nb_streams; i++) {
        //流的类型
        if (pFormatCtx->streams[i]->codec->codec_type == AVMEDIA_TYPE_VIDEO) {
            v_stream_idx = i;
            break;
        }
    }

    if (v_stream_idx == -1) {
        LOGE("%s","can't get input stream\n");
        return;
    }

    //只有知道视频的编码方式，才能够根据编码方式去找到解码器
    //获取视频流中的编解码上下文
    AVCodecContext *pCodecCtx = pFormatCtx->streams[v_stream_idx]->codec;
    //4.根据编解码上下文中的编码id查找对应的解码
    AVCodec *pCodec = avcodec_find_decoder(pCodecCtx->codec_id);
    //（迅雷看看，找不到解码器，临时下载一个解码器）
    if (pCodec == NULL) {
        LOGE("%s","not find Decoder!\n");
        return;
    }

    //5.打开解码器
    if (avcodec_open2(pCodecCtx,pCodec,NULL)<0) {
        LOGE("%s","can't open Decoder!\n");
        return;
    }

    //输出视频信息
    LOGI("视频的文件格式：%s",pFormatCtx->iformat->name);
    LOGI("视频时长：%d", (pFormatCtx->duration)/1000000);
    LOGI("视频的宽高：%d,%d",pCodecCtx->width,pCodecCtx->height);
    LOGI("解码器的名称：%s",pCodec->name);

    //准备读取
    //AVPacket用于存储一帧一帧的压缩数据（H264）
    //缓冲区，开辟空间
    AVPacket *packet = (AVPacket*)av_malloc(sizeof(AVPacket));

    //AVFrame用于存储解码后的像素数据(YUV)
    //内存分配
    AVFrame *pFrame = av_frame_alloc();
    //YUV420
    AVFrame *pFrameYUV = av_frame_alloc();
    //只有指定了AVFrame的像素格式、画面大小才能真正分配内存
    //缓冲区分配内存
    uint8_t *out_buffer = (uint8_t *)av_malloc(avpicture_get_size(AV_PIX_FMT_YUV420P, pCodecCtx->width, pCodecCtx->height));
    //初始化缓冲区
    avpicture_fill((AVPicture *)pFrameYUV, out_buffer, AV_PIX_FMT_YUV420P, pCodecCtx->width, pCodecCtx->height);

    //用于转码（缩放）的参数，转之前的宽高，转之后的宽高，格式等
    struct SwsContext *sws_ctx = sws_getContext(pCodecCtx->width,pCodecCtx->height,pCodecCtx->pix_fmt,
                                                pCodecCtx->width, pCodecCtx->height, AV_PIX_FMT_YUV420P,
                                                SWS_BICUBIC, NULL, NULL, NULL);

    int got_picture, ret;

    FILE *fp_yuv = fopen(output_cstr, "wb+");

    int frame_count = 0;

    //6.一帧一帧的读取压缩数据
    while (av_read_frame(pFormatCtx, packet) >= 0) {
        //只要视频压缩数据（根据流的索引位置判断）
        if (packet->stream_index == v_stream_idx) {
            //7.解码一帧视频压缩数据，得到视频像素数据
            ret = avcodec_decode_video2(pCodecCtx, pFrame, &got_picture, packet);
            if (ret < 0) {
                LOGE("%s","解码错误");
                return;
            }

            //为0说明解码完成，非0正在解码
            if (got_picture) {
                //AVFrame转为像素格式YUV420，宽高
                //2 6输入、输出数据
                //3 7输入、输出画面一行的数据的大小 AVFrame 转换是一行一行转换的
                //4 输入数据第一列要转码的位置 从0开始
                //5 输入画面的高度
                sws_scale(sws_ctx, pFrame->data, pFrame->linesize, 0, pCodecCtx->height,
                          pFrameYUV->data, pFrameYUV->linesize);

                //输出到YUV文件
                //AVFrame像素帧写入文件
                //data解码后的图像像素数据（音频采样数据）
                //Y 亮度 UV 色度（压缩了） 人对亮度更加敏感
                //U V 个数是Y的1/4
                int y_size = pCodecCtx->width * pCodecCtx->height;
                fwrite(pFrameYUV->data[0], 1, y_size, fp_yuv);
                fwrite(pFrameYUV->data[1], 1, y_size / 4, fp_yuv);
                fwrite(pFrameYUV->data[2], 1, y_size / 4, fp_yuv);

                frame_count++;
                LOGI("解码第%d帧",frame_count);
            }
        }

        //释放资源
        av_free_packet(packet);
    }

    //释放资源
    fclose(fp_yuv);
    (*env)->ReleaseStringUTFChars(env,input_jstr,input_cstr);
    (*env)->ReleaseStringUTFChars(env,output_jstr,output_cstr);
    av_frame_free(&pFrame);
    avcodec_close(pCodecCtx);
    avformat_free_context(pFormatCtx);
}
```

然后找一个mp4文件上传到你手机的sd卡目录下 名字叫input.mp4

然后app跑起来点击转码按钮（时间很长）

yuv文件很大，我这里20mb的mp4转换出来是2.7GB

##  源码下载  ##

[源码打包下载](http://gloomyer.com/upload/ubuntu_build_ffmpeg_as_convert.zip)