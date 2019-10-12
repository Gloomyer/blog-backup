---
title: Android差分生成-下-Linux服务器So生成
date: 2017-6-26 22:06:27
categories: Android
tags: 
- 差分生成
---
<Excerpt in index | 首页摘要> 
>  Android差分生成-下-Linux服务器So生成
> <!-- more -->
> <The rest of contents | 余下全文> 

##  概述  ##
因为服务器一般是跑在linux服务器上的

而linuxjni需要的动态库是so不是dll

所以我们需要专门去生成linux下服务器拆分文件所需要的so动态库

##  需要的东西  ##
1>[bzip2](http://gloomyer.com/upload/bzip2.7z)

2>[bsdiff](http://gloomyer.com/upload/bsdiff-4.3.7z)

3>eclipse

4>linux服务器一台(可以是模拟器)

##  流程  ##

首先创建一个目录.

然后解压bzip2把里面所有的.c .h文件放入目录中

然后解压bsdiff，把其中的bsdiff.c放在目录中

打开bsdiff.c 修改main函数名为 bsdiff_main

打开bzip2.c 修改main函数名为 bzip2_main

用eclipse创建一个java项目

首先创建JniUtils.java

内容如下:

```java
package com.gloomyer.demo;

public class JniUtils {
	
	public static native void diff(String oldPath, String newPath, String patchPatch);
	
	static {
		//注意这个路径，这个路径就是保存so文件的全路径 加载全路径利用load，loadlibrary不支持路径
		System.load("/root/bsdiffso/bsdiff.so");
	}
}

```

然后创建Test.java

```java
package com.gloomyer.demo;

public class Test {
	public static void main(String... args){
		JniUtils.diff("v1.apk", "v2.apk", "patch.patch");
	}
}

```

然后进入项目的src目录.打开命令行.

输入:javah JniUtils全路径名称(带包名的那种) 生成.h头文件

把这个生成的头文件拷贝到之前创建的目录里面去

找到你的android ndk目录

里面的platforms里面随便一个android-xx目录的里面随便一个目录下的usr目录->include目录 找到jni.h 

拷贝到刚才创建的目录中

编辑我们生成的那个.h文件

然后编辑里面的include jni 把尖括号换成双引号

编辑bsdiff.c

删除:

```c
#if 0
__FBSDID("$FreeBSD: src/usr.bin/bsdiff/bsdiff/bsdiff.c,v 1.1 2005/08/06 01:59:05 cperciva Exp $");
#endif
```

然后吧include bzlib.h的尖括号改成双引号

然后在文件顶部加入:

```c
#include "com_gloomyer_demo_JniUtils.h"

```

文件末尾加入:

```c
JNIEXPORT void JNICALL Java_com_gloomyer_demo_JniUtils_diff
  (JNIEnv * env, jclass jcls, jstring oldpath_jstr, jstring newpath_jstr, jstring patchpath_jstr){
		char * argv[4] = {"bsdiff"};
		argv[1] = (*env)->GetStringUTFChars(env, oldpath_jstr, NULL);
		argv[2] = (*env)->GetStringUTFChars(env, newpath_jstr, NULL);
		argv[3] = (*env)->GetStringUTFChars(env, patchpath_jstr, NULL);
		bsdiff_main(4, argv);
		
		(*env)->ReleaseStringUTFChars(env, oldpath_jstr, argv[1]);
		(*env)->ReleaseStringUTFChars(env, newpath_jstr, argv[2]);
		(*env)->ReleaseStringUTFChars(env, patchpath_jstr, argv[3]);
  }

  void main(int argc, char * argv){
	  
  }
```

最后的目录结构如下:

![](http://gloomyer.com/img/img/difference_update_03.png)

然后吧整个目录上传至服务器

然后ssh连接至服务器

然后进入刚才上传的目录中

输入:

```shell
gcc -fPIC -shared blocksort.c decompress.c bsdiff.c randtable.c bzip2.c huffman.c compress.c bzlib.c crctable.c  -o bsdiff.so
```

然后将之前用eclipse创建的项目打包成jar，上传至我们刚才上传至服务器的目录中去

然后生成 v1， v2版本的apk取名为v1.apk v2.apk 也上传到刚才那个目录中去

然后在刚才那个目录下执行

```shell
java -jar diff.jar
```

ok 我这里成功的运行了起来。那么说明我这里的so文件是创建正确的。

然后就可以将这个so交给后端使用啦！