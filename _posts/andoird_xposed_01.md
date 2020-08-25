---
title: Android Xposed 开发
date: 2020-01-03 17:31:32
categories: Android
tags:
- Android
---
<Excerpt in index | 首页摘要>
> Android Xposed 开发
<!-- more -->
<The rest of contents | 余下全文>  

## 啥是Xposed
Xposed框架(Xposed Framework)是一套开源的、在Android高权限模式下运行的框架服务，可以在不修改APK文件的情况下影响程序运行(修改系统)的框架服务，基于它可以制作出许多功能强大的模块，且在功能不冲突的情况下同时运作。

xposed的主要作用是hook应用方法，动态劫持方法的运行。

<font color='red'>重点说明，Xposed插件是运行在劫持的APP内部！</font>

## 开发入门
### 项目配置
修改AndroidManifest文件，在application节点下加入
```xml
<meta-data
    android:name="xposedmodule"
    android:value="true"/>
<meta-data
    android:name="xposeddescription"
    android:value="xposed插件描述"/>
<meta-data
    android:name="xposedminversion"
    android:value="30"/>
```

注意这里面的三个meta-data标签的name不能错误，不然xposed框架apk无法识别自定义编写的xposed模块。

第一个是描述这是一个xposed插件，第二个在xposed manager 中对你插件的描述，第三个是表述最低支持的xposed版本。


修改app下的build.gradle 文件,dependencies 下增加
```gradle
compileOnly 'de.robv.android.xposed:api:82'
```

### 开发一个劫持Activity获取启动参数的小插件

创建一个类，实现IXposedHookLoadPackage 接口 类似：
```java
/**
 * @Classname IXposedHookLoadPackage
 * @Description hook
 * @Date 2019-12-13 09:49
 * @Created by gloomy
 */
public class Hook implements IXposedHookLoadPackage {
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam lpParam) throws Throwable {
        XposedHelpers.findAndHookMethod(Activity.class,
            "onCreate", Bundle.class, new XC_MethodHook() {
                @Override
                protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                    super.afterHookedMethod(param);
                    Field mIntentField = Activity.class.getDeclaredField("mIntent");
                    mIntentField.setAccessible(true);
                    Intent intent = (Intent) mIntentField.get(param.thisObject);
                    LG.logE(intent.toUri(Intent.URI_INTENT_SCHEME));
                }
            });

    }
}
```

然后在assets目录下创建xposed_init文件，文件没有后缀名，内容就是上面这个接口的全类名
```txt
com.gloomyer.hook.Hook
```

xposed应用本身不需要提供界面，但是一般插件如果需要动态配置些什么，那么最好你还是些一个界面。

<font color='red'>注意：你提供的界面是在luncher中点开启动的activity，而xposed的代码运行在劫持进程中。所以代码虽然在一个项目中，但是如果需要通信，你需要走跨进程通信！（最简单的方式 发广播。）</font>

上面的功能跑起来以后（记得去Xposed Manager中勾选你的插件，然后重启设备），你可以获取每个activity启动的参数了，如果对方没做什么安全检测。你就可以通过am命令启动任意activity了。

注意每次对插件代码上的改动，你都需要重启设备。