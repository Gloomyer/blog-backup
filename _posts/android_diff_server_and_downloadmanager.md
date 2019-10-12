---
title: Android 日记（2019-10-12）
date: 2019-10-12 17:31:32
categories: Android
tags:
- Android
---
<Excerpt in index | 首页摘要>
> 1.差分升级部署
> 2.更新的时候如何下载APK
<!-- more -->
<The rest of contents | 余下全文>  

## 差分升级部署
初版项目提交，决定将差分升级引入项目，作为基础组件之一。

所以完整的一圈流程，包含服务端，客户端的实现与应用。

这里就不讲代码了，具体看这两个仓库[Android客户端代码](https://github.com/Gloomyer/diff_android_library) [服务端端代码](https://github.com/Gloomyer/bsdiff_server)

服务端代码只支持2种系统（mac/linux）

服务端代码想要跑起来需要安装bzlib

[差分实现原理](https://gloomyer.com/2017/06/26/difference_update_01/)


## DownloadManager

### 介绍

在APP更新中，我们一定要做的事情就是下载新版本的APK安装包。

那么如何下载？ 通常网上流传着大量的DownloadFileUtils。 一般调用okhttp去下载。

但是大部分都没有实现的一件事情：多线程下载。

多线程下载是非常有效的一个提升下载速度的方案，利用多个线程抢占CPU片段的方式来提升速度。

难道就没有使用多线程下载技术的DownloadUtils么？ 是有的，但是大部分都是一个library的方式提交给你。

但是实际上系统已经预置了一个下载管理器。

先说一下优缺点

|功能|框架|系统提供的DownloadManager|
-|-|-
|下载方案|下载做了一定的优化|由系统提供各种优化，小米rom下还会利用迅雷加速|
|引入大小|N个类，超多方法数|实际上是一种跨进程调用，只需要一个类提供对接即可|
|缺点|框架体积庞大/方法数多/如果下载过程中app被杀会导致apk下载失败|一个类，简单，不提供暂停/继续下载功能，回调通知不灵敏，下载过程中用户可以去系统下载管理器暂停任务，但是我们没有办法继续下载|


DownloadManager 由我们通知他启动下载任务，完成活着失败 他会通过广播通知我们（也可以自己查询）

进度回调需要自己查询（没有框架灵敏 小米rom下他是每隔1s 才向内容提供者更新数据，我们只能通过内容提供者获取下载进度）。

### 具体代码

这里没有去注册广播监听完成，因为要做进度查询 已经可以获取到是否完成了。

```java
import android.app.DownloadManager;
import android.content.Context;
import android.database.Cursor;
import android.net.Uri;

import java.io.File;

/**
 * @Classname DownloadFileUtils
 * @Description 文件下载工具类
 * @Date 2019-10-11 17:10
 * @Created by gloomy
 */
public class DownloadFileUtils {

    private Context mContext;
    private File path;
    private Callback callback;
    private DownloadManager downloadManager;
    private long downloadId;

    /**
     * 启动一个下载任务
     *
     * @param context  上下
     * @param url      要下载的文件url
     * @param path     要保存的文件全路径
     * @param callback 回调（回调全部产生于子线程）
     */
    public DownloadFileUtils(Context context,
                             String url,
                             File path,
                             Callback callback) {
        this.mContext = context.getApplicationContext();
        this.path = path;
        this.callback = callback;
        downloadManager = (DownloadManager) mContext.getSystemService(Context.DOWNLOAD_SERVICE);
        downloadAPK(url, path);
    }

    private void downloadAPK(String url, File path) {
        DownloadManager.Request request = new DownloadManager.Request(Uri.parse(url));
        request.setAllowedOverRoaming(false);
        request.setNotificationVisibility(DownloadManager.Request.VISIBILITY_VISIBLE);
        request.setTitle("appName");
        request.setDescription("下载中...");
        request.setVisibleInDownloadsUi(true);
        request.setDestinationUri(Uri.fromFile(path));
        request.setAllowedNetworkTypes(DownloadManager.Request.NETWORK_WIFI
                | DownloadManager.Request.NETWORK_MOBILE);

        //将下载请求加入下载队列，加入下载队列后会给该任务返回一个long型的id，通过该id可以取消任务，重启任务、获取下载的文件等等
        if (downloadManager != null) {
            downloadId = downloadManager.enqueue(request);
        }

        new Thread(queryRunner).start();

    }

    private Runnable queryRunner = new Runnable() {

        private int status;
        private long soFar;
        private long total;
        private int progress;

        @Override
        public void run() {
            while ((soFar < total || total <= 0)
                    && (status == DownloadManager.STATUS_RUNNING
                    || status == DownloadManager.STATUS_PENDING
                    || status == DownloadManager.STATUS_PAUSED
                    || status == 0)) {
                query();
            }

        }

        private void query() {
            DownloadManager.Query query = new DownloadManager.Query();
            query.setFilterById(downloadId);
            Cursor cursor = downloadManager.query(query);
            if (cursor.moveToNext()) {
                status = cursor.getInt(cursor.getColumnIndex(DownloadManager.COLUMN_STATUS));

                soFar = cursor.getLong(
                        cursor.getColumnIndex(DownloadManager.COLUMN_BYTES_DOWNLOADED_SO_FAR));
                if (total <= 0) {
                    total = cursor.getLong(
                            cursor.getColumnIndex(DownloadManager.COLUMN_TOTAL_SIZE_BYTES));
                }

                progress = (int) ((soFar * 100f) / total);

                if (status == DownloadManager.STATUS_PENDING) {
                    if (callback != null)
                        callback.callback(0, soFar, total, progress);
                } else if (status == DownloadManager.STATUS_SUCCESSFUL
                        || status == DownloadManager.STATUS_FAILED) {
                    if (callback != null)
                        callback.callback(
                                status == DownloadManager.STATUS_SUCCESSFUL
                                        ? 2 : 3,
                                soFar, total, progress);
                } else {
                    if (callback != null)
                        callback.callback(0, soFar, total, progress);
                }

            } else {
                status = DownloadManager.STATUS_FAILED;
                callback.callback(3, soFar, total, progress);
            }

            try {
                if(cursor != null) cursor.close();
                Thread.sleep(1000); //1秒才更新一次数据库，这里再小会查询到大量的重复数据
            } catch (Exception ignored) {

            }
        }
    };


    /**
     * 下载回调
     */
    public interface Callback {
        /**
         * 下载回调
         *
         * @param what     0:开始下载 1:下载进度回调, 2:成功,3:失败
         * @param soFar    已经下载的字节数
         * @param total    文件大小
         * @param progress 文件进度（0-100）
         */
        void callback(int what, long soFar, long total, int progress);
    }

}

```