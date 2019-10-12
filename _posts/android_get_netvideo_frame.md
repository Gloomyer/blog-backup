---
title: Android获取网络视频时长、取指定时间帧图片
date: 2018-9-3 17:02:27
categories: Android
tags: 
- Android
---
<Excerpt in index | 首页摘要> 
>  Android获取网络视频时长
>  Android获取网络视频指定时间帧图片
> <!-- more -->
> <The rest of contents | 余下全文> 

## 如何获取
app内经常涉及到视频，如果视频是存放在我们自己的云存储（如七牛）

可以通过url+参数的方式获取我们需要的信息

但是如果是外链视频，要获取一些视频就比较麻烦了

但是Android封装了MediaMetadataRetriever类

来帮助我们获取视频的一些信息

支持网络视频、本地视频

## 具体步骤

先new 对象
```
MediaMetadataRetriever mmr = new MediaMetadataRetriever();
```

然后设置数据源，有多个回调，本地的设置本地视频路径，网络的设置网络视频URL，我这里用网络的

headers 是请求网络视频的请求头内容，可以不加

特别需要注意的是这一步是耗时操作，不会触发异常 但是会导致app卡主，所以需要放在子线程运行
```
HashMap<String, String> headers = new HashMap<>();
headers.put("User-Agent", "Mozilla/5.0 (Linux; U; Android 4.4.2; zh-CN; MW-KW-001 Build/JRO03C) AppleWebKit/533.1 (KHTML, like Gecko) Version/4.0 UCBrowser/1.0.0.001 U4/0.8.0 Mobile Safari/533.1");
mmr.setDataSource(flowInfo.getVideo(), headers);
```

获取视频时长

返回的是字符串类型，毫秒值
```
String duration = mmr.extractMetadata(android.media.MediaMetadataRetriever.METADATA_KEY_DURATION);
```

获取视频的指定时间的帧数，比如我这里的需求是大于5秒的视频取第5秒的帧

小于的取最后一帧
```
Bitmap bitmap;
if (Long.valueOf(duration) > 5000) {
    bitmap = mmr.getFrameAtTime(5000, MediaMetadataRetriever.OPTION_NEXT_SYNC);
} else {
    bitmap = mmr.getFrameAtTime(Long.valueOf(duration), MediaMetadataRetriever.OPTION_NEXT_SYNC);
}
```

拿到之后可以考虑先保存到本地，或者干其他用处，我这里是保存下来咯
```
File file = CacheFileManager.get().getFile("shareStoryImage");
file = new File(file, MD5Utils.encrypt(url) + ".jpg");
FileOutputStream fos = null;
fos = new FileOutputStream(file);
bitmap.compress(Bitmap.CompressFormat.JPEG, 100, fos);
fos.flush();
fos.close();
```