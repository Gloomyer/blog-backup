---
title: Android 图片加水印
date: 2016-7-9 14:48:31
categories: Android
tags:
- Android
- Bitmap
- Canvas
---
<Excerpt in index | 首页摘要> 
> Android 图片加水印  
>
<!-- more -->
<The rest of contents | 余下全文>  
  
项目APP社交类应用，属于富图应用，自然少不了图片下载了~  
  
为了保护连载主的版权信息，在图片下载之后自然需要给图片增加水印图片了。  
  
按理来说，这样的水印应该都是在后台来处理了。  
  
不巧的是我们图片用的是七牛图片处理。我们自己的后台没有图片的备份，包括上传也是没有经过后台的。  
  
七牛自己也带了图片水印的添加，但是我们需要对下载的图片进行一个简单的缩放，而七牛并不支持同时调用两种图片处理  
  
而且我们水印是图片+文字，七牛水印对文字+图片混合水印的支持也不够美好。  
  
最后大任还是交给了我们前端。  
  
--- 
简单说下水印的添加方式，也比较简单，就是把图片读到内存中，然后创建画布，在上面画上原图+水印信息就好了。  
  
直接看代码：  
```java
	/**
	 *	给图片添加水印，返回添加好图片的水印，需要一个已经下载完毕的图片对象
	 */
	public static File Watermark(File file){
		//把要添加水印的图片读入到内存中，注意，这里图片不能太大哟。
		Bitmap bitmap = BitmapFactory.decodeFile(file.getAbsolutePath());

		//创建画布，因为Android不支持对原图的修改，所以创建一个新的bitmap对象来操作
        Bitmap newBitmap = Bitmap.createBitmap(bitmap.getWidth(), bitmap.getHeight(), Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(newBitmap);
        canvas.drawBitmap(bitmap, 0, 0, null);//把原图画作为背景画在新的画布上

		// 把要添加的水印图片读到内存中来
		Bitmap watermarkImg = getWatermarkImg(newBitmap.getWidth() > newBitmap.getHeight()
                ? newBitmap.getHeight() : newBitmap.getWidth());
        Paint paint = new Paint();
        canvas.drawBitmap(watermarkImg, margin
                , (newBitmap.getHeight() - watermarkImg.getHeight() - margin), paint);

		//添加文字水印
        paint.setColor(Color.rgb(0xff, 0xff, 0xff));
        paint.setTextSize(watermarkImg.getHeight() / 4);
        canvas.drawText("要添加的水印文字", margin * 2 + watermarkImg.getWidth(),
                newBitmap.getHeight() - (watermarkImg.getHeight() / 2) - margin, paint);

        //canvas.restore(); android 6.0 Bug 在github上面有人讨论，如果是android 6.0，使用这个会出问题！
        canvas.save(Canvas.ALL_SAVE_FLAG);
		//最后，就是把添加水印完成图片写到sd卡中去了
		try {
            file.delete(); //删除老的没有水印的图片
            FileOutputStream fos = new FileOutputStream(file);
            newBitmap.compress(Bitmap.CompressFormat.JPEG, 100, fos);
            fos.flush();
            fos.close();
        } catch (Exception e) {

        }
        return file;
	}  
  
	/**
     * 获取要加载到图片上的水印图片
     *
     * @return
     */
    private static Bitmap getWatermarkImg(int width) {

        Bitmap imgWatermark = null;
        try {
            imgWatermark = BitmapFactory.decodeStream(
                    MyApp.getInstance().getAssets().open("shuiyin.png"));


            //将水印图片缩放至图片的7/1宽度的高宽
            Matrix matrix = new Matrix();
            float scale = width * 1.0f / 7 / imgWatermark.getWidth();
            matrix.postScale(scale, scale
                    , imgWatermark.getWidth() * 1.0f / 2, imgWatermark.getHeight() * 1.0f / 2);
            imgWatermark =
                    Bitmap.createBitmap(imgWatermark, 0, 0,
                            imgWatermark.getWidth(), imgWatermark.getHeight()
                            , matrix, true);
        } catch (IOException e) {
            imgWatermark = null;
            e.printStackTrace();
        }

        return imgWatermark;
    }  
```
<font color=red>最后，图片添加水印是属于耗时操作,务必在新开的子线程中去操作！！！</font>