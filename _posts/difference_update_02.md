---
title: Android差分升级-中-apk利用差分包合并
date: 2017-6-26 15:43:30
categories: Android
tags: 
- 差分合并
---
<Excerpt in index | 首页摘要> 
>  Android差分升级-中-apk利用差分包合并
> <!-- more -->
> <The rest of contents | 余下全文> 

##  概述  ##
本来这篇打算将合并和linuxso的生成，后发现内容太多，先将合并吧。

首先阅读本篇之前，需要了解差分的生成:[Android差分升级-上-apk差分](http://gloomyer.com/2017/06/26/difference_update_01/)

本篇的源码（web、Android,v1.apk,v2.apk,v1-v2.patch 文章结尾提供下载）

##  流程分析  ##
上篇中，我们知道了如何进行v2.0和v1.0的apk的差分包的生成。

这篇利用bsdiff来利用生成的差分包进行合并成为新版本的apk包并提示安装。

##  工具  ##
1>AS //AndroidStudio

2>[bzip2](http://gloomyer.com/upload/bzip2.7z)

3>[bpatch.c](http://gloomyer.com/upload/bsdiff-4.3.7z)

## 流程 ##
废话不多说。

首先，创建一个包含c++的项目(as这里用的是cmake).

然后打开CMakeLists.txt文件

编辑内容如下:

```txt
cmake_minimum_required(VERSION 3.4.1)

add_library( # Sets the name of the library.
             bspatch

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             src/main/cpp/bspatch.c )

find_library( # Sets the name of the path variable.
              log-lib

              # Specifies the name of the NDK library that
              # you want CMake to locate.
              log )

target_link_libraries( # Specifies the target library.
                       bspatch

                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib} )
```

我删除了注释内容，主要就是这些了.

修改完成之后点击同步按钮

我这里生成了jni目录不知道你们那里生成了没有。如果有删了即可。

删除cpp目录下所有内容。

讲MainActivity里面和jni有关的内容删了。

复制bzip2到cpp目录下。

复制bsdiff里面的bspatch.c到cpp目录下。

项目结构如截图：

![](http://gloomyer.com/img/img/difference_update_02.png)

然后打开bspatch.c

删掉顶部注释，然后修改顶部的include命令
从

```c
#if 0
__FBSDID("$FreeBSD: src/usr.bin/bsdiff/bspatch/bspatch.c,v 1.1 2005/08/06 01:59:06 cperciva Exp $");
#endif

#include <bzlib.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <err.h>
#include <unistd.h>
#include <fcntl.h>
```

改为

```c
#include <jni.h>
#include "bzip2/bzlib.c"
#include "bzip2/crctable.c"
#include "bzip2/compress.c"
#include "bzip2/decompress.c"
#include "bzip2/randtable.c"
#include "bzip2/blocksort.c"
#include "bzip2/huffman.c"

#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <err.h>
#include <unistd.h>
#include <fcntl.h>
```

创建JniUtils.java

内容如下:
```java
package com.gloomyer.diff_update;

public class JniUtils {
    public static native void merge(String oldPath, String patchPath, String newPath);

    static {
        System.loadLibrary("bspatch");
    }
}

```

然后打开bspatch.c文件

修改main函数函数名为bspatch_merge

在文件结尾加入如下实现:

这里说一下bspatch_merge的两个参数

第一个参数恒定为4

第二个参数是个字符串数组,0:无意义随便填,1:老的apk路径(已经安装的应用的apk如何获取安装包 下面的那个ApkUtils里面有方法) 2:生成的新的apk的文件路径 3：差分包路径

```c
JNIEXPORT void JNICALL
Java_com_gloomyer_diff_1update_JniUtils_merge(JNIEnv *env, jclass jcls, jstring oldPath_jstr,
                                              jstring patchPath_jstr, jstring newPath_jstr) {

    char *argv[4] = {"bsdiff"};
    argv[1] = (char *) (*env)->GetStringUTFChars(env, oldPath_jstr, 0);
    argv[2] = (char *) (*env)->GetStringUTFChars(env, newPath_jstr, 0);
    argv[3] = (char *) (*env)->GetStringUTFChars(env, patchPath_jstr, 0);

    bspatch_merge(4, argv);

    (*env)->ReleaseStringUTFChars(env, oldPath_jstr, argv[1]);
    (*env)->ReleaseStringUTFChars(env, newPath_jstr, argv[2]);
    (*env)->ReleaseStringUTFChars(env, patchPath_jstr, argv[3]);
}
```

还有一个工具类:

ApkUtils:

```java
package com.gloomyer.diff_update;

import android.content.Context;
import android.content.Intent;
import android.content.pm.ApplicationInfo;
import android.net.Uri;
import android.text.TextUtils;

/**
 * Created by Gloomy on 2017/6/26.
 */

class ApkUtils {

    public static String getOldPath(Context mContext, String packageName) {
        if (TextUtils.isEmpty(packageName))
            return null;

        try {
            ApplicationInfo appInfo = mContext.getPackageManager()
                    .getApplicationInfo(packageName, 0);
            return appInfo.sourceDir;
        } catch (Exception e) {
            e.printStackTrace();
        }

        return null;
    }



    public static void installApk(Context mContext, String apkPath) {

        Intent intent = new Intent(Intent.ACTION_VIEW);
        intent.setDataAndType(Uri.parse("file://" + apkPath),
                "application/vnd.android.package-archive");
        mContext.startActivity(intent);
    }
}

```

实体类:
```java
package com.gloomyer.diff_update;

/**
 * Created by Gloomy on 2017/6/26.
 */

public class UpdateBean {
    private boolean hasUpdate;
    private String url;

    public boolean isHasUpdate() {
        return hasUpdate;
    }

    public void setHasUpdate(boolean hasUpdate) {
        this.hasUpdate = hasUpdate;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    @Override
    public String toString() {
        return "UpdateBean{" +
                "hasUpdate=" + hasUpdate +
                ", url='" + url + '\'' +
                '}';
    }
}

```

MainActivity代码

主要内容就是根据后台检测更新，然后根据返回结果下载差分包然后调用jni合并成新的包，然后提示用户安装。

```java
package com.gloomyer.diff_update;

import android.os.Environment;
import android.os.Handler;
import android.os.Message;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Toast;

import com.google.gson.Gson;

import java.io.BufferedOutputStream;
import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.net.HttpURLConnection;
import java.net.URL;

public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";
    private static final String BASEURL = "http://192.168.3.213:8090/";
    private static final String requestUrl = BASEURL + "DiffServlet";
    private Handler mHander;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mHander = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                if (msg.what == 0) {
                    downloadPatch((UpdateBean) msg.obj);
                } else if (msg.what == 1) {
                    ApkUtils.installApk(MainActivity.this, (String) msg.obj);
                }
            }

        };
    }

    private void downloadPatch(final UpdateBean info) {
        Log.e(TAG, info.toString());
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    URL url = new URL(info.getUrl());
                    HttpURLConnection connection = (HttpURLConnection) url.openConnection();
                    connection.setDoOutput(true);
                    connection.setDoInput(true);
                    connection.setUseCaches(false);
                    connection.setConnectTimeout(60000);
                    connection.setReadTimeout(60000);

                    File patchFile = Environment.getExternalStorageDirectory();
                    patchFile = new File(patchFile, "patch_" + System.currentTimeMillis() + ".patch");

                    connection.connect();
                    if (connection.getResponseCode() == HttpURLConnection.HTTP_OK) {
                        InputStream is = connection.getInputStream();
                        BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(patchFile));

                        int len;
                        byte buffer[] = new byte[512];
                        while ((len = is.read(buffer)) > 0) {
                            bos.write(buffer, 0, len);
                        }

                        bos.flush();
                        bos.close();
                        is.close();
                    }

                    //合并文件
                    File newApkFile = Environment.getExternalStorageDirectory();
                    newApkFile = new File(newApkFile, "apkfile_" + System.currentTimeMillis() + ".apk");
                    JniUtils.merge(ApkUtils.getOldPath(MainActivity.this, getPackageName()),
                            patchFile.getAbsolutePath(),
                            newApkFile.getAbsolutePath());

                    mHander.obtainMessage(1, newApkFile.getAbsolutePath()).sendToTarget();
                } catch (Exception e) {
                    e.printStackTrace();
                }

            }
        }).start();
    }

    public void update(View v) {

        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    URL url = new URL(requestUrl);
                    HttpURLConnection connection = (HttpURLConnection) url.openConnection();
                    connection.setRequestMethod("GET");
                    connection.setRequestProperty("Charset", "UTF-8");
                    connection.setRequestProperty("Content-Type", "text/xml; charset=UTF-8");
                    connection.connect();
                    if (connection.getResponseCode() == 200) {
                        InputStream is = connection.getInputStream();
                        int len;
                        byte buffer[] = new byte[512];
                        ByteArrayOutputStream baos = new ByteArrayOutputStream();
                        if ((len = is.read(buffer)) > 0) {
                            baos.write(buffer, 0, len);
                        }
                        baos.flush();

                        String json = new String(baos.toByteArray());
                        UpdateBean info = new Gson().fromJson(json, UpdateBean.class);
                        baos.close();
                        is.close();


                        mHander.obtainMessage(0, info).sendToTarget();
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}

```

##  打包下载  ##

这边相对来说比后端要完成的东西要简单一些。

源码我这里打包一下。

[打包下载](http://gloomyer.com/upload/differenceupdatesource.7z)