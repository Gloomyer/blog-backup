---
title: Android 插件化-01
date: 2017-02-28 20:21:54
categories: Android
tags:
- Android
- 插件化
- 组件化
---
<Excerpt in index | 首页摘要> 
> Android 插件化-01
>
> 如何加载apk中的资源
>
> <!-- more -->
> <The rest of contents | 余下全文>  



####  概述  ####

当项目庞大到一定的底部，

往往涉及到一定的并行开发。

比较成熟的解决思路有两种：

1：插件化、组件化开发。

2：h5开发



我希望选择的方向是插件化组件化开发。（因为h5的流畅度还是有待提升）

这不是一篇博客能说完的。本篇主要讲如何加载其他apk中的资源文件。



####  需要了解的预备知识  ####

1、ClassLoader

2、reflect



####    如何加载###

严格来说会碰到两种apk。（已经安装到了手机上apk，和没有安装到apk上的手机）

同样的Android提供给了我们两种ClassLoader。

1：PathClassLoader（用于已经安装完成的apk）

2：DexClassLoader（用于没有安装的apk）



####  预备，走  ####

首先，创建一个项目。

项目本身包含一个model->app

我们在创建一个model->plugin(Phone类型)



然后在两个model的AndroidManifest的根节点都加入shareUserId属性

如下:

```java
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.gloomyer.dex"
    android:sharedUserId="com.gloomyer.myapp">
```

<font color='red'>两个model的包名用不同！！但是这个shareUserId必须一致！</font>

[关于shareUserId](http://www.cnblogs.com/mythou/p/3258715.html)



然后在plugin的mipmap中放入一张图片

然后将plugin运行到我们的手机上。



####  加载已经安装的apk资源  ####

切换至appmodel下

首先创建一个pojo

```java
/**
 * Created by Gloomy on 2017/3/2.
 */
public class Plugin {
    public String packName;
    public String name;

    public Plugin(String name, String packName) {
        this.name = name;
        this.packName = packName;
    }

    public Plugin() {
    }
}
```



然后我们创建一个PluginManager类

添加如下方法

```java
//获取所有已经安装到手机的同shareUserIdapk
//这里的上下文用activity、application都可以
public static List<Plugin> findAllInstalledPlugin(Context mContext) {
        List<Plugin> plugins = new ArrayList<>();
  		//获取所有安装的apk信息
        List<PackageInfo> installedPackages = mContext
                .getPackageManager()
                .getInstalledPackages(PackageManager.MATCH_UNINSTALLED_PACKAGES);
		//遍历这个集合
        for (PackageInfo info : installedPackages) {
            String packName = info.packageName;

            String shareUserId = info.sharedUserId;
			//和当前的shareUserId必须一致，并且包名不一致，才认为是一个插件
            if (shareUserId != null
                    && "com.gloomyer.myapp".equals(shareUserId)
                    && !mContext.getPackageName().equals(packName)) {
              	//获取app的名称
                String name = mContext
                        .getPackageManager()
                        .getApplicationLabel(info.applicationInfo).toString();
                plugins.add(new Plugin(name, packName));
            }
        }
        return plugins;//返回找到的插件集合
}
```

通过上面的方法，我们找到了当前安装在手机上的apk.

现在开始利用classloader加载apk并且反射获取图片id!

```java
//通过一些已知信息 反射图片的id
//重点这里的Context必须得使用createpackageContext来创建！！！
//name是我们图片的名字
public static int dynamicLoadApk(String packName, Context pluginContext, String name) throws Exception {
        PathClassLoader loader
                = new PathClassLoader(pluginContext.getPackageResourcePath(),
                ClassLoader.getSystemClassLoader());
		//反射R文件，R文件全路径为包名+R mipmap是内部类 所以和R连接是$不是.
  		//第二个参数是是否初始化
  		//第三个参数是classloader
        Class<?> clazz = Class.forName(packName + ".R$mipmap", true, loader);
        Field fideld = clazz.getDeclaredField(name);
        return fideld.getInt(R.mipmap.class);
}
```

然后回到MainActivity

```java
 protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
   		ImageView mImageView = (ImageView) findViewById(R.id.mImageView);
   		List<Plugin> plugins = PluginManager.findAllInstalledPlugin(this);
  		if (plugins != null && !plugins.isEmpty()) {
           Plugin plugin = plugins.get(0);//因为我们自己安装的 所以确定只有一个。
           Context pluginContext = createPackageContext(plugin.packName,
                                    CONTEXT_IGNORE_SECURITY | CONTEXT_INCLUDE_CODE);
           //加载apk反射获取图片的id
           int id = PluginManager.dynamicLoadApk(plugin.packName,
                                    pluginContext,
                                    "meinv");/
           //加载图片
           mImageView.setImageDrawable(
                                    pluginContext.getResources().getDrawable(id));
  		}else{
          Toast.makeText(this, "请先安装插件!", Toast.LENGTH_SHORT).show();
  		}
}
```



####  加载未安装apk的资源  ####

首先，卸载之前安装的plugin的app

然后找到plugin的debugapk

在plugin model的 build/outputs/apk/app-debug.apk

然后在这里打开命令行

输入

```shell
adb push app-debug.apk sdcard/plugin.apk
//上传apk到手机sdcard重命名为plugin.apk
```



还是创建一个PluginManager

```java
//读取sd卡上apk的信息
public static Plugin findSdCardApk(Context mContext) {
        PackageManager pm = mContext.getPackageManager();
        PackageInfo packageInfo = pm.getPackageArchiveInfo(
                new File(Environment.getExternalStorageDirectory(), "plugin.apk").getAbsolutePath()
                , PackageManager.GET_ACTIVITIES);
        if (packageInfo != null) {
            String packageName = packageInfo.packageName;
            String shareUserId = packageInfo.sharedUserId;
            if ("com.gloomyer.myapp".equals(shareUserId)
                    && !mContext.getPackageName().equals(packageName)) {
                String name = pm.getApplicationLabel(packageInfo.applicationInfo).toString();
                return new Plugin(name, packageName);
            }
        }
        return null;
}
```



这里和安装过的apk有些不一样。

安装过的apk我们利用createContext可以创建出它的上下文然后获取资源。

但是没安装过的不行。

所以我们需要手动去创建一个Resources，

然后利用AssetManager的addAssetPath方法，将apk上下文环境添加进运行环境

然后利用这个Resources来读取资源



在PluginManager中

```java
//创建资源对象 
public static Resources getUninstallApkRes(Context mContext, Plugin plugin) throws Exception {
  		//反射创建
        AssetManager am = AssetManager.class.newInstance();
  		//这个方法是public的但是是@hide标签，所以得反射调用
        Method method = AssetManager.class.getMethod("addAssetPath", String.class);
        method.invoke(am,
                new File(Environment.getExternalStorageDirectory(), "plugin.apk").getAbsolutePath());
  		//获取当前的资源
        Resources superRes = mContext.getResources();
  		//创建新的资源对象，利用当前的资源对象给予它屏幕参数等等类似的参数。。
        Resources mResources = new Resources(am,
                superRes.getDisplayMetrics(), superRes.getConfiguration());
        return mResources;
    }
```

获取了Resources对象，然后我们继续反射获取id。

PluginManager中

```java

public static int dynamicLoadApk(Context mContext, Plugin plugin, String name) throws Exception {
  		//这个路径是加载之后的dex释放路径
  		//4.x版本之后不允许为sdCard， 我们也最好不要放sd卡(安全性考虑)
        File outputFile = mContext.getDir("dex", Context.MODE_PRIVATE);
  		//sd卡要加载的apk全路径
        File apkFile = new File(Environment.getExternalStorageDirectory(), "plugin.apk");
        DexClassLoader loader
                = new DexClassLoader(apkFile.getAbsolutePath(),
                outputFile.getAbsolutePath(),
                null, //so文件的路径，我们不需要null即可
                ClassLoader.getSystemClassLoader());//系统classloader，通用写法
		
  		//反射获取值
        Class<?> clazz = Class.forName(plugin.packName + ".R$mipmap", true, loader);
        Field field = clazz.getDeclaredField(name);
        return field.getInt(R.mipmap.class);
}
```

然后回到ManActivity

```java
protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
   		ImageView mImageView = (ImageView) findViewById(R.id.mImageView);
   		Plugin plugin = PluginManager.findSdCardApk(this);
  		if (plugin != null) {
            //获取未安装apk的资源对象
            Resources uninstallApkRes = PluginManager
                                .getUninstallApkRes(MainActivity.this, plugin);
           //加载未安装apk反射获取图片的id
           int id = PluginManager.dynamicLoadApk(MainActivity.this, plugin, 							 	"meinv");
           //加载图片
           mImageView.setImageDrawable(
                                    pluginContext.getResources().getDrawable(id));
  		}else{
          Toast.makeText(this, "请先安装插件!", Toast.LENGTH_SHORT).show();
  		}
}
```



End