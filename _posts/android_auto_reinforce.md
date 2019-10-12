---
title: Android 日记
date: 2019-09-20 17:31:32
categories: Android
tags:
- Android
---
<Excerpt in index | 首页摘要>
> 1.gradle 脚本中如何区分正式库与测试库
> 2.如何解决 Android多个渠道包自动加固问题
<!-- more -->
<The rest of contents | 余下全文>  

## gradle 脚本中如何区分正式库与测试库

### 为什么要区分?
是这样的，我们希望在测试库可以通过模拟器进行日常开发（需要x86架构支持）

但是正式库，我们希望对apk进行精简优化，不需要x86架构 只要armeabi-v7a

每次打包去修改非常麻烦， 特别是 如果打好包了又发现没有修改，那叫一个难受。

那么就希望在gradle中去检测正式库和测试库，进行不同的配置 

### 如何区分
直接上代码
```gradle
android{
    ...
    defaultConfig {
        ...
        def isRelease = false
        for (String s : gradle.startParameter.taskNames) {
            if (s.contains("Release") | s.contains("release")) {
                isRelease = true
                break
            }
        }
        //从本地local.properties中读取变量
        def properties = new Properties()
        def inputStream = project.rootProject.file('local.properties').newDataInputStream()
        properties.load(inputStream)

        ndk {
            //默认不包含x86的so库，本地开发时如果要在模拟器上运行，可在local.properties文件中添加：
            //useX86=true
            if (properties.getProperty('useX86') != null
                    && properties.getProperty('useX86').toBoolean()
                    && !isRelease) {
                abiFilters "armeabi-v7a", "x86"
            } else {
                abiFilters "armeabi-v7a"
            }
        }
        ...
    }
    ...
}
```

## 如何解决 Android多个渠道包自动加固问题
### 思路
这里不说代码了（太多了），说一下思路，下面提供github链接

首先通过配置 android gradle 实现多渠道包

然后用脚本去遍历app目录，获取生成好的apk文件。 

然后调用腾讯乐固提供的api，将文件上传到乐固进行加固，

遍历查询结果，ok之后下载加固包。

利用Android提供的签名工具进行对apk的重新签名。

这里说一下，因为乐固调用api不支持直接上传apk文件， 所以需要运维/后端 在云存储方面提供支持。

我现在的公司用的是七牛的服务，运维专门给我开了一个bocket，用来存储apk文件。

所以为了加固，我会将未加固的apk上传到七牛。

为了方便给测试和产品给包。我也会将加固签名好后的apk包传到七牛云，最后还会将apk的实际下载地址生成二维码，方便他们下载。

具体的使用方法，在github 里面有，这里就不详细描述了

### github地址
[android 自动打包 自动加固工具](https://github.com/Gloomyer/AutoReinforce)
