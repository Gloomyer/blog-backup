---
title: 利用Fmod模仿QQ魔音效果
date: 2017-6-30 13:42:18
categories: Android
tags: 
- ndk
- fmod
---
<Excerpt in index | 首页摘要> 
> 利用Fmod模仿QQ魔音效果
> <!-- more -->
> <The rest of contents | 余下全文> 

##  前提  ##
本文将讲解如何利用Fmod来实现qq的魔音效果。

（没有QQ的魔音效果好，因为本人不懂音律方面的知识，参数配置也是按照别人来的，所以效果没有QQ的好。）

继续阅读之前，你可以先看一下[AndroidStudio利用ndk-build编译fmod](http://gloomyer.com/2017/06/29/as_build_fmod_01/)

文章结束，提供源码下载
##  工具  ##
1->as

2->ndk

3->fmod

##  正文  ##
首先创建一个空的Android工程.

gradle.properties中加入如下语句:

```
android.useDeprecatedNdk=true
```

app的build.gradle中android节点下加入:

```
sourceSets {
    main {
        jni.srcDirs = []
        jniLibs.srcDirs = ['src/main/libs']
    }
}
```


在app/src/main目录下创建目录jni目录、assets目录、libs目录

然后录制一段话，随便说些什么，把录音文件放在assets目录下.

打开fmod文件夹

打开api->lowlevel-inc目录，将里面的所有文件拷贝纸jni目录下

打开api->lowlevel->libs目录

将fmod.jar复制到app\libs目录下

打开armeabi-v7文件夹，将里面的so文件复制到jni目录下

找到你的ndk安装目录

然后找到platforms->android-??(随意)->arch-arm->usr->include目录下 找到jni.h 也拷贝至jni目录下

Activity的布局文件:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:orientation="vertical"
    android:layout_height="match_parent">

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="playSound0"
        android:text="普通播放" />

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="playSound1"
        android:layout_marginTop="10dp"
        android:text="萝莉模式播放" />

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="playSound2"
        android:layout_marginTop="10dp"
        android:text="大叔模式播放" />

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="playSound3"
        android:layout_marginTop="10dp"
        android:text="搞怪播放" />

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="playSound4"
        android:layout_marginTop="10dp"
        android:text="空灵效果播放" />

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="playSound5"
        android:layout_marginTop="10dp"
        android:text="惊悚效果播放" />
</LinearLayout>

```


Activity代码:

```
package org.fmod.example;

import android.os.Bundle;
import android.app.Activity;
import android.view.View;

public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        org.fmod.FMOD.init(this);
    }


    public void playSound0(View v) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                playSound("sound.mp3", 0);
            }
        }).start();
    }

    public void playSound1(View v) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                playSound("sound.mp3", 1);
            }
        }).start();
    }

    public void playSound2(View v) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                playSound("sound.mp3", 2);
            }
        }).start();
    }

    public void playSound3(View v) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                playSound("sound.mp3", 3);
            }
        }).start();
    }

    public void playSound4(View v) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                playSound("sound.mp3", 4);
            }
        }).start();
    }

    public void playSound5(View v) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                playSound("sound.mp3", 5);
            }
        }).start();
    }


    @Override
    protected void onDestroy() {
        org.fmod.FMOD.close();
        super.onDestroy();
    }


    public static native void playSound(String fileName, int type);

    static {
        System.loadLibrary("playsound");
    }
}

```

打开terminal窗口，进入到app/src/main/java目录下

输入

```
javah org.fmod.example.MainActivity //Activity的全路径
```

生成头文件，刷新一下 然后将生成的头文件移动至jni目录下.

打开生成的头文件，将include jni.h 的尖括号换成双引号(不然没有jni的代码提示)

myplaysound.cpp代码:

```
//
// Created by Gloomy on 2017/6/30.
//

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include "org_fmod_example_MainActivity.h"
#include "fmod.hpp"
#include "fmod_dsp_effects.h"

using namespace FMOD;

JNIEXPORT void JNICALL Java_org_fmod_example_MainActivity_playSound
        (JNIEnv *env, jclass jcls, jstring filename_jstr, jint type) {
    const char *filename = env->GetStringUTFChars(filename_jstr, NULL);
    System *system;
    Sound *sound;
    Channel *channel;
    DSP *dsp;
    bool isPlaying = true;


    System_Create(&system);
    system->init(32, FMOD_INIT_NORMAL, NULL);

    char *filePath = (char *) calloc(256, sizeof(char));
    strcat(filePath, "file:///android_asset/");
    strcat(filePath, filename);

    try {
        system->createSound(filePath, FMOD_DEFAULT, 0, &sound);
        system->playSound(sound, 0, false, &channel);
        switch (type) {
            case 0://正常
                break;
            case 1://萝莉
                system->createDSPByType(FMOD_DSP_TYPE_PITCHSHIFT, &dsp); //萝莉
                dsp->setParameterFloat(FMOD_DSP_PITCHSHIFT_PITCH, 2.5f);
                channel->addDSP(0, dsp);
                break;
            case 2://大叔
                system->createDSPByType(FMOD_DSP_TYPE_PITCHSHIFT, &dsp);
                dsp->setParameterFloat(FMOD_DSP_PITCHSHIFT_PITCH, 0.8f);
                channel->addDSP(0, dsp);
                break;
            case 3://搞怪
                float frequency;
                channel->getFrequency(&frequency);
                frequency *= 2.0f;
                channel->setFrequency(frequency);
                break;
            case 4: //空灵
                system->createDSPByType(FMOD_DSP_TYPE_ECHO, &dsp);
                dsp->setParameterFloat(FMOD_DSP_ECHO_DELAY, 300);
                dsp->setParameterFloat(FMOD_DSP_ECHO_FEEDBACK, 20);
                channel->addDSP(0, dsp);
                break;
            case 5://惊悚
                system->createDSPByType(FMOD_DSP_TYPE_TREMOLO, &dsp);
                dsp->setParameterFloat(FMOD_DSP_TREMOLO_SKEW, 0.5f);
                channel->addDSP(0, dsp);
                break;
        }
    } catch (...) {
        goto end;
    }

    system->update();
    do {
        channel->isPlaying(&isPlaying);
        usleep(1000 * 1000);
    } while (isPlaying);

    end:
    free(filePath);
    sound->release();
    system->close();
    system->release();

    env->ReleaseStringUTFChars(filename_jstr, filename);
}

```

这些声音效果的配置是看别人的我也不太懂。如果有想要自定义，可以打开fmod_dsp_effects.h 去看里面的备注.

在jni目录下创建Android.mk Application.mk

Android.mk:

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)
LOCAL_MODULE    := fmod
LOCAL_SRC_FILES := libfmod.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE    := fmodL
LOCAL_SRC_FILES := libfmodL.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE    := playsound
LOCAL_SRC_FILES := myplaysound.cpp
LOCAL_SHARED_LIBRARIES  := fmod fmodL
LOCAL_CPP_FEATURES  := exceptions
include $(BUILD_SHARED_LIBRARY)
```

Application.mk:

```
APP_ABI := armeabi-v7a
APP_STL := gnustl_static
```

打开terminal窗口，路径保证在app\src\main下。

输入ndk-build(如果没有配置去配置ndk环境变量)

等待成功生成so文件.然后直接run项目到手机上就好啦!

##  源码下载  ##

[源码打包下载](http://gloomyer.com/upload/as_imitationqq_beats_by_fmod.7z)