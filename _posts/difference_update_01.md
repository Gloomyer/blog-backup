---
title: Android差分升级-上-apk差分
date: 2017-6-26 10:11:29
categories: Android
tags: 
- 差分升级
---
<Excerpt in index | 首页摘要> 
> Android差分升级-上-apk差分
> <!-- more -->
> <The rest of contents | 余下全文> 

##  什么是差分升级  ##

在APP日益增大的体积下，每次更新都是一次痛苦的旅程。

特别在集成视频等功能后，体积很容易到30mb以上。

每次功能上的更新都需要用户去浪费30mb的网络去下载。

虽然现在很多都是wifi居多，但是有些用户一看体积太大就不愿意去更新了。

差分升级就是为了应对这种场景，比如我有2个app：v1.0，v2.0.

v2.0是v1.0的升级包,v1.0 30mb， v2.032mb。

那么我们只需要找到这两个包的差异值，生成差分包，客户端只需要下载这个差分包来升级即可。

##  流程分析  ##
这是需要后台互相配合，方可实现的一套功能点。

双方都需要运用到jni技术.

这篇讲服务器的差分生成.

##  需要的工具  ##
1.visual studio(编译Windows下运用到的dll文件)

2.eclipse(开发web)

3.[bsdiff(差分分析生成patch的一个第三方框架)](http://gloomyer.com/upload/bsdiff4.3-win32-src.zip)


##  实践  ##
首先生成web端需要jni开发调用的dll文件.

打开visualstudio.创建一个空的项目

copybsdiff4.3-win32-src里面的所有.c/.cpp/.h文件(不包含bspatch.cpp)到项目中

结构图如下

![](http://gloomyer.com/img/img/difference_update_01.png)

然后修改下配置:

项目菜单->BsDiff(项目名称)属性->配置属性->配置类型(修改为动态库.dll)

项目菜单->BsDiff(项目名称)属性->配置属性->c/c++->常规->sdl检查(修改为否)

然后如果在vs2011的版本以上，还需要配置

项目菜单->BsDiff(项目名称)属性->配置属性->c/c++->命令行

编辑框输入

```
-D _CRT_SECURE_NO_DEPRECATE -D _CRT_NONSTDC_NO_DEPRECATE 
```

然后确定.

最后，修改编译平台为x64（我的电脑系统是64的）(这个如果不会自行百度一下，很简单的配置)

打开bsdiff.cpp

这个是主文件，找到入口main函数.

这个东西原来是生成.exe程序，然后根据命令行参数实现的。现在我们需要通过jni调用就要修改一下。

首先修改一下main名字为bsdiff_diff。

说一下这个函数的调用参数,第一个参数argc恒定为4.第二个argv是个字符串数组，size为4，0:无效随意填，1：老的apk路径，2:新apk的路径 3:patch包的路径

打开eclipse创建一个web项目.(我这里用的idea，原理一样的)

然后创建一个BsDiffUtils工具类

代码如下:

```java
public class BsDiffUtils {
    public static native void diff(String oldPath, String newPath, String patchPath);

    static {
        System.loadLibrary("BsDiff");
    }
}
```

然后进入web项目的src目录.

利用javah命令+刚才的BsDiffutils的全路径生成.h文件

然后找到你的jdk安装目录，在里面的include目录下找到jni.h。

还有/include/win32/目录下 找到jni_md.h

总共三个头文件(com_gloomyer_demo_BsDiffUtils.h(利用javah生成的头文件),jni.h,jni_md.h),copy至vs的项目里面

然后添加现有项目，把这三个头文件添加至项目中.

修改我们生成的com_gloomyer_demo_BsDiffUtils.h的

```
#include <jni.h>
```
为

```
#include "jni.h"
```

然后打开bsdiff.cpp 

添加#include "com_gloomyer_demo_BsDiffUtils.h"

然后copy com_gloomyer_demo_BsDiffUtils.h里面的函数声明到bsdiff.cpp文件末尾

实现它，修改成如下内容:

```c
JNIEXPORT void JNICALL Java_com_gloomyer_demo_BsDiffUtils_diff
(JNIEnv * env, jclass jcls, jstring oldfile_jstr, jstring newfile_jstr, jstring patchfile_jstr){
	char *argv[4] = {"BsDiff"};
	argv[1] = (char *)env->GetStringUTFChars(oldfile_jstr, NULL);
	argv[2] = (char *)env->GetStringUTFChars(newfile_jstr, NULL);
	argv[3] = (char *)env->GetStringUTFChars(patchfile_jstr, NULL);
	bsdiff_diff(4, argv);

	env->ReleaseStringUTFChars(oldfile_jstr, argv[1]);
	env->ReleaseStringUTFChars(newfile_jstr, argv[2]);
	env->ReleaseStringUTFChars(patchfile_jstr, argv[3]);
}
```

然后点击生成，然后copy生成好的dll文件到web项目的根目录下.

然后创建一个Test类，尝试调用jni。测试一下。

生成两个apk，一个新的一个久的。

```java
public class Test {
    public static void main(String... args) {
        BsDiffUtils.diff("D:\\old.apk", "D:\\new.apk", "D:\\patch.patch");
    }
}
```

然后跑起来，拆分是一个耗时操作，如果两个apk差异越大，时间越长。

好了 因为我的v1.0和v2.0就几个图片的差距所以很快

old.apk:1.5mb

new.apk:3.75mb

patch.patch 3.12mb

完美。下一篇将讲如何生成Linux的so包供web调用还有android拿到patch后如何合并.