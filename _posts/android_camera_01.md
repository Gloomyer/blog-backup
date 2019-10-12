---
title: Android调用系统相机照相
date: 2016-7-21 15:54:46
categories: Android
tags:
- Android
- 相机
- camera
---
<Excerpt in index | 首页摘要> 
> Android调用系统相机照相  
>
<!-- more -->
<The rest of contents | 余下全文>  
  
项目APP创建一个“阶段”的时候有添加图片的需求，自然是让用户可以自由选择照相还是选取已经存在于相册的图片了  
  
调用系统相册问题不大。就不细说了  
  
主要说一下调用照相机。  
  
首先你需要配置权限  
```xml  
<uses-permission android:name="android.permission.CAMERA"/>  
```
然后通过隐式意图启动相机  
```java
Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
intent.setType("image/*");
// 开启一个带有返回值的Activity，请求码为PHOTO_REQUEST_GALLERY
startActivityForResult(intent, 321);  //请求码自定义
```
重点来了，Android原生的设定是返回一个内容提供者Uri，然后由我们自己解析获取到图片。  
  
但是国内Android都是深度定制过的。  
  
个别机型修改了这个设置，目前发现的：小米、魅族  
  
他们是在返回的数据中直接携带了图片的真实全路径那么我们就需要做一个简单的适配  
```java
if (data.getData() != null) {
	//如果能够直接读到这个数据，就说明直接拿到了图片的真实路径保存路径即可
    String file = data.getData().toString();
    imageBgPath = file;
} else {
	//如果拿不到，就说明需要我们通过内容解析者去解析获取
	//以下是标准格式
    Bitmap bm = null;
    ContentResolver resolver = getContentResolver();

    String[] proj = {MediaStore.Images.Media.DATA};

    Cursor cursor = managedQuery(data.getData(), proj, null, null, null);

    int column_index;
    try {
        column_index = cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DATA);

    } catch (Exception e) {
        column_index = 0;
    }

    cursor.moveToFirst();

    imageBgPath = cursor.getString(column_index);
}
L.i(TAG, "path:  " + imageBgPath);
```