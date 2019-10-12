---
title: 国产rom开发中碰到的几个坑
date: 2017-11-29 16:21:12
categories: Android
tags: 
- Android
---
<Excerpt in index | 首页摘要> 
> 反射创建对象失败,提示没有权限访问
> 
> 锤子ROM一启动app就crash
> 
> <!-- more -->
> <The rest of contents | 余下全文> 

###  反射创建对象失败,提示没有权限访问  ###
平台:flyme、锤子

简单的说，当前项目是抽取ViewModel层,activity负责UI方面和传递相应数据给viewholder的工作

viewholder负责具体逻辑、类似网络请求等操作。

然后新版本测试的时候发现，两个界面进入显示空UI 没有具体的数据（消息、个人信息）

而且目前发现有这个问题的只有flyme、锤子等2个rom。通过logcat发现 日志提示内容是没有权限创建viewholder的对象(viewholder对象的创建是依赖反射来创建的)

最开始误以为是因为appcrash后rom提供的classloader不一致导致的。

后来发现是我想太多。

真实的原因是那两个界面的viewholderclass的访问修饰符是default的！

但是鬼知道flyme、锤子做了什么操作 activity和viewholder在一个包下的情况 它认为没有权限！而且奇葩的事情是 直接run上去的app没有问题，如果是利用apk安装出来的就有锅。

有一句mmp，不知当讲不当讲！

###  锤子ROM一启动app就crash  ###
产品这两天一直反馈我提供的安装包安装完成的app一打开就crash

通过bugly统计日志发现错误信息如下:
```
SIGSEGV(SEGV_MAPERR)
#00 pc 000000000029d31c /system/lib64/libart.so (_ZN3art6mirror9ArtMethod7ToDexPcEmb+220) [arm64-v8a::8f291f51cb11a01a56d5cb19d9514401]
2 #01 pc 0000000000308e48 /system/lib64/libart.so (_ZN3art20CurrentMethodVisitor10VisitFrameEv+76) [arm64-v8a::8f291f51cb11a01a56d5cb19d9514401]
3 #02 pc 0000000000307bf0 /system/lib64/libart.so (_ZN3art12StackVisitor9WalkStackEb+180) [arm64-v8a::8f291f51cb11a01a56d5cb19d9514401]
4 #03 pc 0000000000311e50 /system/lib64/libart.so (_ZNK3art6Thread4DumpERNSt3__113basic_ostreamIcNS1_11char_traitsIcEEEE+264) [arm64-v8a::8f291f51cb11a01a56d5cb19d9514401]
5 #04 pc 000000000031ab90 /system/lib64/libart.so (_ZN3art10ThreadList10DumpLockedERNSt3__113basic_ostreamIcNS1_11char_traitsIcEEEE+100) [arm64-v8a::8f291f51cb11a01a56d5cb19d9514401]
6 #05 pc 00000000002fa69c /system/lib64/libart.so (_ZN3art7Runtime5AbortEv+572) [arm64-v8a::8f291f51cb11a01a56d5cb19d9514401]
7 #06 pc 00000000000cce90 /system/lib64/libart.so (_ZN3art10LogMessageD1Ev+2748) [arm64-v8a::8f291f51cb11a01a56d5cb19d9514401]
8 #07 pc 00000000000dc044 /system/lib64/libart.so (_ZN3artL8JniAbortEPKcS1_+2132) [arm64-v8a::8f291f51cb11a01a56d5cb19d9514401]
9 #08 pc 00000000000dc858 /system/lib64/libart.so (_ZN3art9JniAbortFEPKcS1_z+216) [arm64-v8a::8f291f51cb11a01a56d5cb19d9514401]
10 #09 pc 00000000000e178c /system/lib64/libart.so (_ZN3art11ScopedCheck5CheckEbPKcz.constprop.108+4924) [arm64-v8a::8f291f51cb11a01a56d5cb19d9514401]
11 #10 pc 00000000000ed814 /system/lib64/libart.so (_ZN3art8CheckJNI12NewStringUTFEP7_JNIEnvPKc+76) [arm64-v8a::8f291f51cb11a01a56d5cb19d9514401]
12 #11 pc 00000000000e39dc /system/lib64/libandroid_runtime.so [arm64-v8a::b8b55f744ba3a9d3a47b2cfdd3a2a6de]
13 #12 pc 000000000039c6f4 /data/dalvik-cache/arm64/system@framework@boot.oat [::f3d88fb28fbfd84f03a796d2eea24c58]
14 java:
15 android.content.res.StringBlock.get(StringBlock.java:82)
16 android.content.res.XmlBlock$Parser.getPooledString(XmlBlock.java:458)
17 android.content.res.TypedArray.loadStringValueAt(TypedArray.java:991)
18 android.content.res.TypedArray.getText(TypedArray.java:144)
19 android.widget.TextView.<init>(TextView.java:956)
20 android.widget.TextView.<init>(TextView.java:666)
21 android.support.v7.widget.AppCompatTextView.<init>(AppCompatTextView.java:75)
22 android.support.v7.widget.AppCompatTextView.<init>(AppCompatTextView.java:71)
23 android.support.v7.app.AppCompatViewInflater.createView(AppCompatViewInflater.java:103)
24 android.support.v7.app.AppCompatDelegateImplV9.createView(AppCompatDelegateImplV9.java:1024)
25 android.support.v7.app.AppCompatDelegateImplV9.onCreateView(AppCompatDelegateImplV9.java:1081)
26 android.view.LayoutInflater.createViewFromTag(LayoutInflater.java:725)
27 android.view.LayoutInflater.rInflate(LayoutInflater.java:806)
28 android.view.LayoutInflater.rInflate(LayoutInflater.java:809)
29 android.view.LayoutInflater.rInflate(LayoutInflater.java:809)
30 android.view.LayoutInflater.rInflate(LayoutInflater.java:809)
31 android.view.LayoutInflater.inflate(LayoutInflater.java:504)
32 android.view.LayoutInflater.inflate(LayoutInflater.java:414)
33 com.miaozan.xpro.base.BaseFragment.onCreateView(BaseFragment.java:44)
34 com.miaozan.xpro.base.MainFragment.onCreateView(MainFragment.java:36)
35 android.support.v4.app.Fragment.performCreateView(Fragment.java:2343)
36 android.support.v4.app.FragmentManagerImpl.moveToState(FragmentManager.java:1419)
37 android.support.v4.app.FragmentManagerImpl.moveFragmentToExpectedState(FragmentManager.java:1740)
38 android.support.v4.app.FragmentManagerImpl.moveToState(FragmentManager.java:1809)
39 android.support.v4.app.BackStackRecord.executeOps(BackStackRecord.java:799)
40 android.support.v4.app.FragmentManagerImpl.executeOps(FragmentManager.java:2580)
41 android.support.v4.app.FragmentManagerImpl.executeOpsTogether(FragmentManager.java:2367)
42 android.support.v4.app.FragmentManagerImpl.removeRedundantOperationsAndExecute(FragmentManager.java:2322)
43 android.support.v4.app.FragmentManagerImpl.execSingleAction(FragmentManager.java:2199)
44 android.support.v4.app.BackStackRecord.commitNowAllowingStateLoss(BackStackRecord.java:651)
45 android.support.v4.app.FragmentPagerAdapter.finishUpdate(FragmentPagerAdapter.java:145)
46 android.support.v4.view.ViewPager.populate(ViewPager.java:1236)
47 android.support.v4.view.ViewPager.populate(ViewPager.java:1084)
48 android.support.v4.view.ViewPager.onMeasure(ViewPager.java:1614)
49 android.view.View.measure(View.java:17600)
50 android.view.ViewGroup.measureChildWithMargins(ViewGroup.java:5589)
51 android.widget.FrameLayout.onMeasure(FrameLayout.java:436)
52 android.view.View.measure(View.java:17600)
53 android.view.ViewGroup.measureChildWithMargins(ViewGroup.java:5589)
54 android.widget.FrameLayout.onMeasure(FrameLayout.java:436)
55 android.support.v7.widget.ContentFrameLayout.onMeasure(ContentFrameLayout.java:139)
56 android.view.View.measure(View.java:17600)
57 android.view.ViewGroup.measureChildWithMargins(ViewGroup.java:5589)
58 android.widget.LinearLayout.measureChildBeforeLayout(LinearLayout.java:1436)
59 android.widget.LinearLayout.measureVertical(LinearLayout.java:722)
60 android.widget.LinearLayout.onMeasure(LinearLayout.java:613)
61 android.view.View.measure(View.java:17600)
62 android.view.ViewGroup.measureChildWithMargins(ViewGroup.java:5589)
63 android.widget.FrameLayout.onMeasure(FrameLayout.java:436)
64 android.view.View.measure(View.java:17600)
65 android.view.ViewGroup.measureChildWithMargins(ViewGroup.java:5589)
66 android.widget.LinearLayout.measureChildBeforeLayout(LinearLayout.java:1436)
67 android.widget.LinearLayout.measureVertical(LinearLayout.java:722)
68 android.widget.LinearLayout.onMeasure(LinearLayout.java:613)
69 android.view.View.measure(View.java:17600)
70 android.view.ViewGroup.measureChildWithMargins(ViewGroup.java:5589)
71 android.widget.FrameLayout.onMeasure(FrameLayout.java:436)
72 com.android.internal.policy.impl.PhoneWindow$DecorView.onMeasure(PhoneWindow.java:2800)
73 android.view.View.measure(View.java:176
74 [Java stack is truncated for it exceeds the max size.]
```
可以明确的发现是ndk方面出问题了。

并且排除了是当前项目中所有的so

最后定位在crash的本地在代码是在fragment创建view的代码

```
@Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
            return cententView = LayoutInflater.from(getContext())
                    .inflate(getLayoutId(), container, false);
 }
```

正常来说这句话是不会出问题的。

tmd，仔细观察了日志 发现是crateString方法抛了异常(native层)

然后去查了下 SIGSEGV(SEGV_MAPERR) 是操作系统认为进行了不安全的操作 抛的指令。

然后仔细想了想  产品前两天要求我在xml里面一个数据为空页面的图片更换成了emoji之后就一直反馈他的手机启动不了app了。

然后将在xml里面直接写死的emoji更换到代码里面去动态设置，果然就好了！

锤子不知道对art虚拟机做了什么奇葩的设定，导致认为在解析xml的时候 如果xml里面包含emoji表情 则认为是不安全的行为，然后对当进程进行了kill操作。

有一句mmp，<SPAN style="TEXT-DECORATION: line-through">不知当讲不当讲！</SPAN>，一定要讲！
