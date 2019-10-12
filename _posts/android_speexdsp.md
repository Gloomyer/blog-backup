---
title: Android 编译speexdsp 对声音进行降噪/回声消除
date: 2019-04-29 17:31:32
categories: Android
tags:
- Android
- JNI
- speex
---
<Excerpt in index | 首页摘要>
> 编译Speexdsp
>
> 在Android平台进行降噪/回声消除处理
>
> <!-- more -->
> <The rest of contents | 余下全文>  

## 概要

*Speex*是一套主要针对语音的开源免费，无专利保护的音频压缩格式

SpeexDsp是一套主要针对语音的噪音屏蔽/回声消除的处理框架.

我们这里主要讲speexdsp 目前最新版本是1.2rc3

[官网speexdsp1.2rc3下载](http://downloads.xiph.org/releases/speex/speexdsp-1.2rc3.tar.gz)

[speex官网 www.speex.org](www.speex.org)





## 编译speexdsp

需要准备

- ndk

- Linux/Mac系统
- speexdsp源码



首先下载源码，解压之后进入到源码根目录执行

```shell
./configure
```



等待一会 等待speexdsp 配置完成。

然后在speexdsp根目录创建Android.mk文件 内容如下

```makefile
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE    := libspeexdsp
LOCAL_CFLAGS = -DFIXED_POINT -DUSE_KISS_FFT -DEXPORT="" -DHAVE_STDINT_H -UHAVE_CONFIG_H
LOCAL_C_INCLUDES := $(LOCAL_PATH)/include
LOCAL_LDLIBS := -llog
LOCAL_SRC_FILES :=  \
./libspeexdsp/buffer.c \
./libspeexdsp/fftwrap.c \
./libspeexdsp/filterbank.c \
./libspeexdsp/jitter.c \
./libspeexdsp/kiss_fft.c \
./libspeexdsp/kiss_fftr.c \
./libspeexdsp/mdf.c \
./libspeexdsp/preprocess.c \
./libspeexdsp/resample.c \
./libspeexdsp/scal.c \
./libspeexdsp/smallft.c

include $(BUILD_SHARED_LIBRARY)
#include $(BUILD_STATIC_LIBRARY)
```

继续创建Application.mk

```makefile
APP_PLATFORM = android-16
APP_ABI := all
```

这里说明一下，如果你需要打armeabi的包 那么这里的最小版本需要修改到9 然后你的ndk也不能特别的新 可以考虑r10

然后执行build命令(需要配置好ndk)

```
ndk-build NDK_PROJECT_PATH=. NDK_APPLICATION_MK=Application.mk APP_BUILD_SCRIPT=Android.mk
```

我这里用的是19 最新的ndk需要指定个mk文件 老版本直接ndk-build即可

## 配置开发环境

将speexdsp include目录copy到cpp开发目录下

将libs生成的so copy 到jniLibs目录下

修改speexdsp_config_types.h文件

```c
#ifndef __SPEEX_TYPES_H__
#define __SPEEX_TYPES_H__

#include "stdint.h"

typedef int16_t spx_int16_t;
typedef uint16_t spx_uint16_t;
typedef int32_t spx_int32_t;
typedef uint32_t spx_uint32_t;

#endif
```

删除.mk .in 多余文件

将CMakeLists.txt移动至app目录下

修改build.gradle

```
android{
	...
	 externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
            version "3.10.2"
        }
    }
}
```

修改CMakeLists.txt内容

```cmake
cmake_minimum_required(VERSION 3.4.1)

add_library(myspeex SHARED src/main/cpp/myspeex.cpp)

target_include_directories(myspeex PRIVATE ${CMAKE_SOURCE_DIR}/src/main/cpp/include/)

add_library(speexdsp SHARED IMPORTED)
set_target_properties(speexdsp PROPERTIES IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/src/main/jniLibs/${CMAKE_ANDROID_ARCH_ABI}/libspeexdsp.so)

find_library(log-lib log)

target_link_libraries(myspeex ${log-lib} speexdsp)
```



## 实现降噪

### java层代码

创建SpeexJNIBridge.java

```java
public class SpeexJNIBridge {

    static {
        System.loadLibrary("speexdsp");
        System.loadLibrary("myspeex");
    }

    public static native void init(int frame_size, int sampling_rate);

    public static native void destory();

    public static native int denoise(byte[] buffer);

    /**
     * 回声消除处理
     * 这个方法包含降噪处理，无需在单独调用
     *
     * @param inBuffer   本地录制的声音(准备发出去的)
     * @param playBuffer 本地准备播放的声音(接受到的)
     * @return 处理好的数据
     */
    public static native byte[] cancellation(byte[] inBuffer, byte[] playBuffer);

```



准备利用AudioRecord录制声音

```java
public class MainActivity extends AppCompatActivity {
   //来源：麦克风
    private final static int AUDIO_INPUT = MediaRecorder.AudioSource.MIC;
    // 采样率
    // 44100是目前的标准，但是某些设备仍然支持22050，16000，11025
    // 采样频率一般共分为22.05KHz、44.1KHz、48KHz三个等级
    private final static int AUDIO_SAMPLE_RATE = 44100;
    // 音频通道 单声道
    private final static int AUDIO_CHANNEL = AudioFormat.CHANNEL_IN_STEREO;
    // 音频格式：PCM编码
    private final static int AUDIO_FORMAT = AudioFormat.ENCODING_PCM_16BIT;
  
   /**
     * 是否启用jni层处理功能
     *
     * @param denoise
     */
    private void record(boolean denoise) {
        if (isRecord) {
            Toast.makeText(this, "请先停止", Toast.LENGTH_SHORT).show();
            return;
        }
        bufferSize = AudioRecord.getMinBufferSize(AUDIO_SAMPLE_RATE,
                AUDIO_CHANNEL, AUDIO_FORMAT);
        mAudioRecord = new AudioRecord(AUDIO_INPUT, AUDIO_SAMPLE_RATE,
                AUDIO_CHANNEL, AUDIO_FORMAT, bufferSize);
        mAudioTrack = new AudioTrack(AudioManager.STREAM_MUSIC, AUDIO_SAMPLE_RATE,
                AUDIO_CHANNEL, AUDIO_FORMAT, bufferSize, AudioTrack.MODE_STREAM);

        if (denoise) {
            SpeexJNIBridge.init(bufferSize, AUDIO_SAMPLE_RATE);
        }

        buffer = new byte[bufferSize];
        if (mAudioRecord.getState() != AudioRecord.STATE_INITIALIZED) {
            mAudioRecord = null;
            Toast.makeText(this, "初始化失败!", Toast.LENGTH_SHORT).show();
            return;
        }

        mAudioRecord.startRecording();
        mAudioTrack.play();
        new Thread(new RecordTask(denoise)).start();
    }
  
   private class RecordTask implements Runnable {

        private final boolean denoise;

        public RecordTask(boolean denoise) {
            this.denoise = denoise;

        }

        @Override
        public void run() {
            Thread.currentThread().setPriority(Thread.MAX_PRIORITY);
            isRecord = true;
            while (isRecord) {
                int read = mAudioRecord.read(buffer, 0, bufferSize);
                if (read >= 2) {
                    if (denoise) {
                        SpeexJNIBridge.denoise(buffer);
                    }
                    mAudioTrack.write(buffer, 0, read);
                }
            }

            if (denoise)
                SpeexJNIBridge.destory();
        }
    }
}
```

这里说一下，因为这是demo，不涉及到发送接收数据，这里没办法测试回声消除（会导致自己的声音消失）

### c/c++代码

就一个文件

myspeex.cpp

```c++
#include <jni.h>
#include <string>
#include <speex/speex_preprocess.h>
#include <speex/speex_echo.h>
#include <android/log.h>

#define LOGE(...) __android_log_print(ANDROID_LOG_ERROR, "gloomy", __VA_ARGS__)

SpeexPreprocessState *state = nullptr;
SpeexEchoState *echoState = nullptr;

extern "C"
JNIEXPORT void JNICALL
Java_com_gloomyer_myspeex_interfaces_SpeexJNIBridge_init(JNIEnv *env, jclass type,
                                                         jint frame_size, jint sampling_rate) {

    frame_size /= 2;
    int filterLength = frame_size * 8;

    //降噪初始化
    state = speex_preprocess_state_init(frame_size, sampling_rate);
    int i = 1;
    speex_preprocess_ctl(state, SPEEX_PREPROCESS_SET_DENOISE, &i); //降噪 建议1
    i = -25;
    speex_preprocess_ctl(state, SPEEX_PREPROCESS_SET_NOISE_SUPPRESS, &i);////设置噪声的dB
    i = 1;
    speex_preprocess_ctl(state, SPEEX_PREPROCESS_SET_AGC, &i);////增益
    i = 24000;
    speex_preprocess_ctl(state, SPEEX_PREPROCESS_SET_AGC_LEVEL, &i);
    i = 0;
    speex_preprocess_ctl(state, SPEEX_PREPROCESS_SET_DEREVERB, &i);
    float f = 0;
    speex_preprocess_ctl(state, SPEEX_PREPROCESS_SET_DEREVERB_DECAY, &f);
    f = 0;
    speex_preprocess_ctl(state, SPEEX_PREPROCESS_SET_DEREVERB_LEVEL, &f);

    //回音消除初始化
    echoState = speex_echo_state_init(frame_size, filterLength);
    speex_echo_ctl(echoState, SPEEX_ECHO_SET_SAMPLING_RATE, &sampling_rate);

    //关联
    speex_preprocess_ctl(state, SPEEX_PREPROCESS_SET_ECHO_STATE, echoState);
}

extern "C"
JNIEXPORT void JNICALL
Java_com_gloomyer_myspeex_interfaces_SpeexJNIBridge_destory(JNIEnv *env, jclass type) {
    if (state) {
        speex_preprocess_state_destroy(state);
        state = nullptr;
    }

    if (echoState) {
        speex_echo_state_destroy(echoState);
        echoState = nullptr;
    }
}

extern "C"
JNIEXPORT jint JNICALL
Java_com_gloomyer_myspeex_interfaces_SpeexJNIBridge_denoise(JNIEnv *env, jclass jcls,
                                                            jbyteArray buffer_) {
    jbyte *in_buffer = env->GetByteArrayElements(buffer_, nullptr);

    auto *in = (short *) in_buffer;
    int rcode = speex_preprocess_run(state, in);

    env->ReleaseByteArrayElements(buffer_, in_buffer, 0);
    return rcode;
}

extern "C"
JNIEXPORT jbyteArray JNICALL
Java_com_gloomyer_myspeex_interfaces_SpeexJNIBridge_cancellation(JNIEnv *env, jclass jcls,
                                                                 jbyteArray inBuffer_,
                                                                 jbyteArray playBuffer_) {
    jbyte *inBuffer = env->GetByteArrayElements(inBuffer_, nullptr);
    jbyte *playBuffer = env->GetByteArrayElements(playBuffer_, nullptr);

    jsize length = env->GetArrayLength(playBuffer_);
    jbyteArray retBuffer_ = env->NewByteArray(length);
    jbyte *retBuffer = env->GetByteArrayElements(retBuffer_, nullptr);

    auto *in = (short *) inBuffer;
    auto *play = (short *) playBuffer;
    auto *ret = (short *) retBuffer;
    speex_echo_cancellation(echoState, in, play, ret); //消除回声
    speex_preprocess_run(state, ret);

    env->ReleaseByteArrayElements(inBuffer_, inBuffer, 0);
    env->ReleaseByteArrayElements(playBuffer_, playBuffer, 0);
    env->ReleaseByteArrayElements(retBuffer_, retBuffer, 0);
    return retBuffer_;
}
```



## 代码下载

[github浏览下载](https://github.com/Gloomyer/speexdsp_android_demo)