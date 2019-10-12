---
title: Android 插件化-02
date: 2017-03-03 17:42:34
categories: Android
tags:
- Android
- 插件化
- 组件化
---
<Excerpt in index | 首页摘要> 
> Android 插件化-02
>
> 如何启动第三方apk的Activity1-反射
>
> <!-- more -->
> <The rest of contents | 余下全文>  



####  概述  ####

上一次说到，如何引用第三方apk的资源文件。

既然解决了资源问题，那么我们如何启动第三方apk中的界面呢?



本篇文章的启动方式是存在缺陷的！需要在宿主app中注册插件的所有要启动的界面，这违背了并行开发的初衷！



适用于：

​	插件中只有一个界面的情况，例如：简书主体和记录模块。



#####  实例  ####

启动第三方apk中的界面，难点就是一个。如何赋予其生命。

我们知道Activity有自己的生命周期。

不是你单纯的反射出class然后new出来就能够使用的。

那么如何赋予我们第三方apk中Activity生命周期呢？



LoadedApk

这个类是在android.app包下的一个类

```
Local state maintained about a currently loaded .apk.
//LoadedApk对象是APK文件在内存中的表示
```

这个类有个成员变量

```
private ClassLoader mClassLoader;
```

这个变量代表加载的classloader。

如果我们能修改这是个对象，那么，我们就可以让activity活起来！

但是我们怎么获取这个类呢？

反射能获取class可是还是我们new出来的，即便获取到了也没有什么意义。



ActivityThread中有个成员变量

```
final ArrayMap<String, WeakReference<LoadedApk>> mPackages
        = new ArrayMap<String, WeakReference<LoadedApk>>();
```

这里存放的就是LoadedApk的实例



也就是说我们需要获取ActivityThread的实例(他自己有个成员变量存储了自身)



然后我们就可以获取到这个实例



####  代码  ####

首先、创建2个model app&plugin phone类型

<font color='red'>(请注意，不可以继承自AppCpmpatActivity 否则会出现各种问题！！)</font>

plugin里面就是一个TextView的界面显示一个字符串.

plugin MainAct代码

//因为发现反射启动这个插件，setContentView会失效。所以通过宿主app填充view然后利用静态方法传递过来

```
 @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        if (contentView == null) {
            setContentView(R.layout.activity_new);
        } else {
            setContentView(contentView);
        }
        contentView = null;

    }

    private static View contentView;


    public static void setLayoutView(View contentView) {
        MainActivity.contentView = contentView;
    }
```

然后我们build plugin的apk 上传至 手机至sdcard 名字叫plugin.apk



在app的model下清单中注册plugin的activity(这里会报红因为找不到,但是没关系不影响运行)



app代码

```java
 @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        findViewById(R.id.mButton).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                startApkAct();
            }
        });
}

 	/**
     * 反射启用第三方apk
     */
    @SuppressLint("NewApi")
    private void startApkAct() {
        //找到插件apk
        String apkFile = new File(Environment.getExternalStorageDirectory(), "plugin.apk").getAbsolutePath();
      	//dex 输出目录
        String outFile = getDir("dex", MODE_PRIVATE).getAbsolutePath();

        //加载资源
      	//这里着重说明下
      	//因为要这样反射调用 发现插件的activity的setContentView 无效，所以需要在这里填充布局
      	//反射设置给插件app
      	//布局又在插件里面
      	//所以需要创建apk的资源读取器
      	//然后重写getResource和getAssets
		//这部分代码在尾部
        loadResources(apkFile);

        //获取classLoader
      	//注意这里的父classLoader一定要用getClassLoader 因为是需要用当前宿主apk的pathclassloader
        DexClassLoader mDexClassLoader
                = new DexClassLoader(apkFile, outFile, outFile, getClassLoader());

        //获取要启动的class和布局id
        Class<?> targetClass = null;
        try {
            targetClass = mDexClassLoader.loadClass("com.gloomyer.plugin.MainActivity");
            Method setLayoutViewMethod = targetClass.getMethod("setLayoutView", View.class);

            //获取布局ID
            Class<?> targetLayoutClass = mDexClassLoader.loadClass("com.gloomyer.plugin.R$layout");
            Field layoutIdField = targetLayoutClass.getField("activity_new");
            Integer layoutId = (Integer) layoutIdField.get(null);
            View contentView = LayoutInflater.from(this).inflate(layoutId, null);

            //反射调用setLayoutView
            setLayoutViewMethod.invoke(null, contentView);

        } catch (Exception e) {
            e.printStackTrace();
        }

        //准备启动
      	//修改LoadedApk的mClassLoader
        try {
            //获取currentActivityThread
            Class<?> ActivityThreadClass = mDexClassLoader.loadClass("android.app.ActivityThread");
            Method currentActivityThreadMethod = ActivityThreadClass.getMethod("currentActivityThread");
            Object currentActivityThread = currentActivityThreadMethod.invoke(null);

            //获取mPackages
            Field mPackagesField = ActivityThreadClass.getDeclaredField("mPackages");
            mPackagesField.setAccessible(true);
            ArrayMap mPackages = (ArrayMap) mPackagesField.get(currentActivityThread);

            //获取loadedApk的实例对象
            WeakReference wr = (WeakReference) mPackages.get(getPackageName());

            //修改loadedApk的loadclass
            Class<?> LoadedApkClass = mDexClassLoader.loadClass("android.app.LoadedApk");
            Field mClassLoaderField = LoadedApkClass.getDeclaredField("mClassLoader");
            mClassLoaderField.setAccessible(true);
            mClassLoaderField.set(wr.get(), mDexClassLoader);
        } catch (Exception e) {

        }


        //启动Activity
        startActivity(new Intent(this, targetClass));
    }


//以下内容是为了使用插件中的资源
protected AssetManager mAssetManager;
    protected Resources mResources;
    protected Theme mTheme;

    private void loadResources(String apkFile) {
        try {
            AssetManager assetManager = AssetManager.class.newInstance();
            Method addAssetPath = assetManager.getClass().getMethod("addAssetPath", String.class);
            addAssetPath.invoke(assetManager, apkFile);
            mAssetManager = assetManager;
        } catch (Exception e) {
            Log.i("inject", "loadResource error:" + Log.getStackTraceString(e));
            e.printStackTrace();
        }
        Resources superRes = super.getResources();
        superRes.getDisplayMetrics();
        superRes.getConfiguration();
        mResources = new Resources(mAssetManager, superRes.getDisplayMetrics(), superRes.getConfiguration());
        mTheme = mResources.newTheme();
        mTheme.setTo(super.getTheme());
    }


    @Override
    public AssetManager getAssets() {
        return mAssetManager == null ? super.getAssets() : mAssetManager;
    }

    @Override
    public Resources getResources() {
        return mResources == null ? super.getResources() : mResources;
    }

    @Override
    public Theme getTheme() {
        return mTheme == null ? super.getTheme() : mTheme;
    }

```



