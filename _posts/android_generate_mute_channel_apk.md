---
title: AndroidSutdio3.0利用友盟实现多渠道打包
date: 2018-10-16 14:55:59
categories: Android
tags:
- Android
- gradle
---
<Excerpt in index | 首页摘要>
> AndroidSutdio3.0利用友盟实现多渠道打包
>
<!-- more -->
<The rest of contents | 余下全文>

## 概要
运营统计开始需要统计不同的渠道app用户数量，和安装app来源的占比关系。
所以就需要在安装包上做手脚。多渠道打包就是用来解决这个问题的。

多渠道打包实现其实是比较容易的，方案也不只一个，目前来说应该有两种类型的方式。一种是在打包的时候打不同渠道的包（我们采用这种方式），还有一种是利用apk包做手脚。

前者慢等于一次要生成N个包，假设一个包生成时间是1分钟，那么10个包就是10分钟。后者快，因为是生成一个包，然后对这个包做手脚。10个包一般也就1分钟+5-6秒的时间。但是前者相对安全。

我们没有那么多的渠道需求，前者相对来说比较好。

## 配置
首先引入友盟统计sdk

然后按照友盟的文档配置好。

在AndroidManifest文件种有这么一条。

```
<meta-data android:name="UMENG_CHANNEL" android:value="${UMENG_CHANNEL_VALUE}" />
```

value 按照这个格式填写，里面的内容自定义 后面需要用。

打开app/build.gradle文件

<font color='red'>在android节点下，defaultConfig节点的versionCode、versionName 后加入</font>
```
flavorDimensions "default"
```
AndroidStudio3.x 需要

### 在android节点下配置签名

```
 signingConfigs {
    release {
        v1SigningEnabled true
        v2SigningEnabled true
        storeFile file("../keystore/keystore.jks")
        storePassword "######"
        keyAlias "####"
        keyPassword "######"
    }
    debug {
        storeFile file("../keystore/keystore.jks")
        storePassword "######"
        keyAlias "####"
        keyPassword "######"
    }
}
```

### 在android节点下配置debug、release包配置
```
buildTypes {
    debug {
        minifyEnabled false
        zipAlignEnabled true
        jniDebuggable true

        signingConfig signingConfigs.debug
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        buildConfigField "boolean", "APP_DEBUG", "true"
    }
    release {
        minifyEnabled false
        zipAlignEnabled true

        signingConfig signingConfigs.release
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        buildConfigField "boolean", "APP_DEBUG", "false"
    }
}
```

### 在android节点下配置渠道
```
productFlavors {
    xpro {
    }
    huawei {
    }
    oppo {
    }
    tencent {
    }
    vivo {
    }
    oneplus {
    }
    smartisan {
    }
    baidu {
    }
    qihu360 {
    }
    xiaomi {
    }
    wandoujia {
    }
    meizu {
    }
}

productFlavors.all {
    flavor -> flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name]
}
```

### 在android节点下配置生成的apk文件名(可选)

```
applicationVariants.all { variant ->
    variant.outputs.all { output ->
        outputFileName = "unreinforced_name:${defaultConfig.versionName}_code:${defaultConfig.versionCode}_${variant.productFlavors[0].name}.apk"
    }
}
```

### 生成包

#### 命令方式
在项目根目录下
多渠道打包命令
打所有debug
./gradlew assembleDebug
打所有release
./gradlew assembleRelease
打单个debug 比如豌豆荚
./gradlew assembleWandoujiaDebug
打单个release 比如豌豆荚
./gradlew assembleWandoujiaRelease

我是Linux 命令执行规则请按照自己系统

#### 图形化操作
在工具栏找到build
build->generate signed bundle/APK->选择APK Next->配置签名 Next->选择apk输出目录 -> 选择build type -> 选择要打的渠道（多选）-> 勾选v1&v2 -> 点击finish
