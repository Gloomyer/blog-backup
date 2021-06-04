---
title: Android app 行为录像
date: 2021-06-04 14:30:00
categories: Android
tags:
- Android
---
<Excerpt in index | 首页摘要> 
> android app 如何录制行为录像  
>
<!-- more -->
<The rest of contents | 余下全文>  

### 原因

虽然app实现了捕捉崩溃日志的功能,但是在bug复现过程中 还是产生了不少的苦难

在接触h5之后, 知晓他们可以通过记录dom事件 来实现录像

突然感觉app实现录像也没有那么困难

视频本质就是图片的集合,我们是可以给自己的app实现截图功能的

<font color='red'>开启录像后cpu 稳定会吃20%左右 所以切记只能在测试环境开启</font>

### 实现方案

启动一个后台service 通过ActivityLifecycleCallbacks 获取栈顶activity, 然后走 drawingCache 获取界面截图

始终保存N（自定义）数量的图片路径　发生崩溃通过广播通知　录像管理器　来将之前缓存的图片封装成mp4视频

### 代码

ErrorRecode.kt
```kotlin
object ErrorRecode {

    var application: Application? = null

    //在 application 主线程中初始化一次
    @Synchronized
    fun onCreate(app: Application) {
        if (application == null) {
            application = app
            app.registerActivityLifecycleCallbacks(ActivityLifecycle)
            try {
                app.startService(Intent(app, ErrorRecodeService::class.java))
            } catch (e: Exception) {

            }
        }
    }
}
```

ActivityLifecycle.kt
```kotlin
@SuppressLint("StaticFieldLeak")
internal object ActivityLifecycle : Application.ActivityLifecycleCallbacks {

    private var resume = false
    private var currentActivity: Activity? = null

    fun isResume(): Boolean = resume

    fun getCurrentActivity(): Activity? = currentActivity

    override fun onActivityCreated(activity: Activity, savedInstanceState: Bundle?) {

    }

    override fun onActivityStarted(activity: Activity) {
    }

    override fun onActivityResumed(activity: Activity) {
        resume = true
        currentActivity = activity
    }

    override fun onActivityPaused(activity: Activity) {
        resume = false
        if (currentActivity == activity) currentActivity = null
    }

    override fun onActivityStopped(activity: Activity) {
    }

    override fun onActivitySaveInstanceState(activity: Activity, outState: Bundle) {
    }

    override fun onActivityDestroyed(activity: Activity) {
    }
}
```

//负责生成图片 和记录
ErrorRecodeService.kt
```kotlin
internal class ErrorRecodeService : IntentService("ErrorRecodeService") {

    companion object {
        private const val MAX_FILE_LENGTH = 5 * 10
        private const val SLEEP_TIME = 200L
        private val fileList = ArrayList<String?>()
    }

    private object ErrorRecodeReceiver : BroadcastReceiver() {

        override fun onReceive(context: Context, intent: Intent) {
            val dataList = ArrayList<String?>()
            //错误日志监听广播接收者,崩溃统计方需要发广播通知这个接收者开始生成视频
            Thread {
                synchronized(ErrorRecodeService::class.java) {
                    dataList.addAll(fileList)
                    fileList.clear()
                }
                VideoUtils.store(context, dataList)
            }.start()
            Log.i("ErrorRecodeService", "onReceive intent:$dataList")
        }

    }

    var destroy = false

    override fun onCreate() {
        super.onCreate()
        val filter = IntentFilter("${application.packageName}.error.recode.receiver")
        application.registerReceiver(ErrorRecodeReceiver, filter)
    }

    override fun onDestroy() {
        super.onDestroy()
        destroy = true
    }

    override fun onHandleIntent(intent: Intent?) {
        var stateBarHeight = 0
        var windowHeight = 0
        var windowWidth = 0
        deleteCache()
        Log.i("ErrorRecodeService", "onHandleIntent")
        while (!destroy) {
            val startTime = System.currentTimeMillis()
            if (ActivityLifecycle.isResume()) {
                val activity = ActivityLifecycle.getCurrentActivity() ?: continue
                val decorView = activity.window?.decorView ?: continue

                if (windowHeight == 0 || windowWidth == 0 || stateBarHeight == 0) {
                    try {
                        val rect = Rect()
                        decorView.getWindowVisibleDisplayFrame(rect)
                        stateBarHeight = rect.top
                        val display = activity.windowManager.defaultDisplay
                        windowWidth = display.width
                        windowHeight = display.height
                    } catch (e: Throwable) {

                    }
                }

                val file = createFile(activity)

                try {
                    if (windowHeight == 0 || windowWidth == 0 || stateBarHeight == 0) continue
                    decorView.isDrawingCacheEnabled = true
                    val bitmap =
                        Bitmap.createBitmap(
                            decorView.drawingCache,
                            0, stateBarHeight,
                            windowWidth,
                            windowHeight,
                        )
                    decorView.isDrawingCacheEnabled = false


                    var fis: FileInputStream?
                    var fos: FileOutputStream?

                    val ret =
                        bitmap.compress(Bitmap.CompressFormat.JPEG, 75, FileOutputStream(file))
                    if (ret) {
                        @Suppress("RECEIVER_NULLABILITY_MISMATCH_BASED_ON_JAVA_ANNOTATIONS")
                        val lbFile = Luban.with(applicationContext)
                            .load(file)
                            .ignoreBy(75)
                            .setTargetDir(file.parentFile.absolutePath)
                            .get()
                        file.delete()
                        if (!lbFile.isNullOrEmpty()) {
                            fis = FileInputStream(lbFile[0])
                            fos = FileOutputStream(file)
                            val buffer = ByteArray(4096)
                            var len: Int
                            try {
                                while (fis.read(buffer).also { len = it } >= 0) {
                                    for (i in 0 until len) {
                                        buffer[i] = buffer[i].xor(0xF)
                                    }
                                    fos.write(buffer, 0, len)
                                }

                                synchronized(ErrorRecodeService::class.java) {
                                    if (fileList.size >= MAX_FILE_LENGTH) {
                                        application.sendBroadcast(
                                            Intent("${application.packageName}.error.recode.receiver").putExtra(
                                                "error",
                                                "error"
                                            )
                                        )
                                        val delFile = fileList.removeAt(0)
                                        if (delFile != null) File(delFile).delete()
                                    }
                                    fileList.add(file.absolutePath)
                                }
                            } catch (e: Exception) {
                            } finally {
                                if (lbFile.size > 0 && lbFile[0] != null) lbFile[0].delete()
                                try {
                                    fos.close()
                                } catch (e: Exception) {
                                }
                                try {
                                    fis.close()
                                } catch (e: Exception) {
                                }
                            }
                        }
                        Log.i("ErrorRecodeService", "save once frame time: ${System.currentTimeMillis() - startTime} ms")
                        continue
                    }
                } catch (e: Throwable) {
                    file.delete()
                }
            }
            Thread.sleep(SLEEP_TIME)
        }
    }

    private fun deleteCache() {
        var file = applicationContext.getExternalFilesDir("error_recode")
        if (file == null) file = applicationContext.cacheDir!!
        file = File(file, "frames")

        if (!file.exists() || file.isFile) file.mkdirs()
        val fileList = file.listFiles() ?: return
        for (f in fileList) {
            try {
                f.delete()
            } catch (e: Throwable) {
            }
        }
    }

    private fun createFile(activity: Activity): File {
        var file = activity.getExternalFilesDir("error_recode")
        if (file == null) file = activity.cacheDir!!
        file = File(file, "frames")

        if (!file.exists() || file.isFile) file.mkdirs()
        return File(file, "${System.currentTimeMillis()}.frame")
    }
}
```


//图片合成视频
VideoUtils.kt
```kotlin
internal object VideoUtils {

    fun getMediaCodecList(): IntArray? {
        //获取解码器列表
        val numCodecs = MediaCodecList.REGULAR_CODECS
        var codecInfo: MediaCodecInfo? = null
        var i = 0
        while (i < numCodecs && codecInfo == null) {
            val info = MediaCodecList.getCodecInfoAt(i)
            if (!info.isEncoder) {
                i++
                continue
            }
            val types = info.supportedTypes
            var found = false
            //轮训所要的解码器
            var j = 0
            while (j < types.size && !found) {
                if (types[j] == "video/avc") {
                    found = true
                }
                j++
            }
            if (!found) {
                i++
                continue
            }
            codecInfo = info
            i++
        }
        Log.i("ErrorRecodeVideoUtils", "found " + codecInfo?.name + " supporting video/avc")
        val capabilities = codecInfo?.getCapabilitiesForType("video/avc")
        return capabilities?.colorFormats
    }

    fun createMediaFormat(activity: Activity): MediaFormat? {
        val decorView = activity.window?.decorView ?: return null
        val rect = Rect()
        decorView.getWindowVisibleDisplayFrame(rect)
        val display = activity.windowManager.defaultDisplay
        val width = display.width
        val height = display.height


        val mediaFormat =
            MediaFormat.createVideoFormat(MediaFormat.MIMETYPE_VIDEO_AVC, width, height);
        mediaFormat.setInteger(
            MediaFormat.KEY_COLOR_FORMAT,
            MediaCodecInfo.CodecCapabilities.COLOR_FormatYUV420Flexible
        )
        mediaFormat.setInteger(MediaFormat.KEY_BIT_RATE, width * height)
        mediaFormat.setInteger(MediaFormat.KEY_FRAME_RATE, 10)
        mediaFormat.setInteger(MediaFormat.KEY_I_FRAME_INTERVAL, 5)
        return mediaFormat
    }

    /**
     * 保存视频文件
     */
    fun store(context: Context, dataList: ArrayList<String?>) {
        val activity = ActivityLifecycle.getCurrentActivity()
        Log.i("ErrorRecodeVideoUtils", "store ThreadName:${Thread.currentThread().name}")
        if (activity != null) {
            getMediaCodecList()
            val mediaFormat = createMediaFormat(activity)
            if (mediaFormat != null) {
                val file = createVideoFile(activity)
                try {
                    writeVideo(mediaFormat, dataList, file)
                } catch (e: Throwable) {
                    file.delete()
                }
            }
        }
        dataList.forEach { path ->
            if (path != null) File(path).delete()
        }
    }

    private fun createVideoFile(activity: Activity): File {
        var file = activity.getExternalFilesDir("error_recode")
        if (file == null) file = activity.cacheDir!!
        file = File(file, "error_aac")

        if (!file.exists() || file.isFile) file.mkdirs()
        return File(file, "${System.currentTimeMillis()}.mp4")
    }

    private fun writeVideo(mediaFormat: MediaFormat, dataList: ArrayList<String?>, file: File) {
        try {
            val mediaCodec = MediaCodec.createEncoderByType(MediaFormat.MIMETYPE_VIDEO_AVC);
            val mediaMuxer =
                MediaMuxer(file.absolutePath, MediaMuxer.OutputFormat.MUXER_OUTPUT_MPEG_4)

            mediaCodec.configure(mediaFormat, null, null, MediaCodec.CONFIGURE_FLAG_ENCODE)
            mediaCodec.start()

            writeVideoFrame(dataList, mediaCodec, mediaMuxer, mediaFormat)

            mediaCodec.stop()
            mediaCodec.release()

            mediaMuxer.stop()
            mediaMuxer.release()
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }

    private fun writeVideoFrame(
        dataList: ArrayList<String?>,
        mediaCodec: MediaCodec,
        mediaMuxer: MediaMuxer,
        mediaFormat: MediaFormat
    ) {
        val mediaCodecBufferInfo = MediaCodec.BufferInfo()

        var generateIndex = 0
        dataList.forEach { path ->
            if (path != null) {
                val inputBufferIndex: Int = mediaCodec.dequeueInputBuffer(10000)
                if (inputBufferIndex > 0) {
                    val bitmap = readBitmap(path) ?: return@forEach
                    val ptsUsec =
                        132L + generateIndex * 1000000 / mediaFormat.getInteger(MediaFormat.KEY_FRAME_RATE)
                    val yuvArray = covertBitmap(bitmap)
                    val inputBuffer = mediaCodec.getInputBuffer(inputBufferIndex)
                    inputBuffer?.clear()
                    inputBuffer?.put(yuvArray)
                    mediaCodec.queueInputBuffer(inputBufferIndex, 0, yuvArray.size, ptsUsec, 0)
                    bitmap.recycle()
                    drainEncoder(false, mediaCodecBufferInfo, mediaCodec, mediaMuxer)
                    generateIndex++
                }
            }
        }

        val inputBufferIndex = mediaCodec.dequeueInputBuffer(10000)
        if (inputBufferIndex >= 0) {
            val ptsUsec =
                132L + generateIndex * 1000000 / mediaFormat.getInteger(MediaFormat.KEY_FRAME_RATE)
            mediaCodec.queueInputBuffer(
                inputBufferIndex, 0, 0, ptsUsec,
                MediaCodec.BUFFER_FLAG_END_OF_STREAM
            )
            drainEncoder(true, mediaCodecBufferInfo, mediaCodec, mediaMuxer)
        }
    }

    private var mMuxerStarted = false
    private var mTrackIndex = 0

    /**
     * 编辑编码器
     */
    private fun drainEncoder(
        isEnd: Boolean,
        mediaCodecBufferInfo: MediaCodec.BufferInfo,
        mediaCodec: MediaCodec,
        mediaMuxer: MediaMuxer
    ) {
        if (isEnd) {
            try {
                mediaCodec.signalEndOfInputStream()
            } catch (e: Exception) {
            }
        }

        while (true) {
            val encoderStatus = mediaCodec.dequeueOutputBuffer(mediaCodecBufferInfo, 10000L)
            if (encoderStatus == MediaCodec.INFO_TRY_AGAIN_LATER) {
                if (!isEnd) {
                    break // out of while
                } else {
                    Log.i(
                        "ErrorRecodeVideoUtils",
                        "no output available, spinning to await EOS"
                    )
                }
            } else if (encoderStatus == MediaCodec.INFO_OUTPUT_FORMAT_CHANGED) {
                if (mMuxerStarted) {
                    throw RuntimeException("format changed twice")
                }
                val mediaFormat = mediaCodec.outputFormat
                mTrackIndex = mediaMuxer.addTrack(mediaFormat)
                mediaMuxer.start()
                mMuxerStarted = true
            } else if (encoderStatus < 0) {
                Log.i(
                    "ErrorRecodeVideoUtils",
                    "unexpected result from encoder.dequeueOutputBuffer: $encoderStatus"
                )
            } else {
                val outputBuffer = mediaCodec.getOutputBuffer(encoderStatus)
                    ?: throw java.lang.RuntimeException(
                        "encoderOutputBuffer "
                                + encoderStatus + " was null"
                    )
                if ((mediaCodecBufferInfo.flags and MediaCodec.BUFFER_FLAG_CODEC_CONFIG) != 0) {
                    Log.d(
                        "ErrorRecodeVideoUtils",
                        "ignoring BUFFER_FLAG_CODEC_CONFIG"
                    )
                    mediaCodecBufferInfo.size = 0
                }
                if (mediaCodecBufferInfo.size != 0) {
                    if (!mMuxerStarted) {
                        throw java.lang.RuntimeException("muxer hasn't started")
                    }

                    // adjust the ByteBuffer values to match BufferInfo
                    outputBuffer.position(mediaCodecBufferInfo.offset)
                    outputBuffer.limit(mediaCodecBufferInfo.offset + mediaCodecBufferInfo.size)
                    Log.d(
                        "ErrorRecodeVideoUtils",
                        ("BufferInfo: " + mediaCodecBufferInfo.offset + ","
                                + mediaCodecBufferInfo.size + ","
                                + mediaCodecBufferInfo.presentationTimeUs)
                    )
                    try {
                        mediaMuxer.writeSampleData(mTrackIndex, outputBuffer, mediaCodecBufferInfo)
                    } catch (e: java.lang.Exception) {
                        Log.i("ErrorRecodeVideoUtils", "Too many frames")
                    }
                }
                mediaCodec.releaseOutputBuffer(encoderStatus, false)
                if ((mediaCodecBufferInfo.flags and MediaCodec.BUFFER_FLAG_END_OF_STREAM) != 0) {
                    if (!isEnd) {
                        Log.i(
                            "ErrorRecodeVideoUtils",
                            "reached end of stream unexpectedly"
                        )
                    } else {
                        Log.i("ErrorRecodeVideoUtils", "end of stream reached")
                    }
                    break // out of while
                }
            }
        }
    }

    /**
     * 图片转成 yuv数组
     */
    private fun covertBitmap(bitmap: Bitmap): ByteArray {

        val inputWidth = bitmap.width / 4 * 4
        val inputHeight = bitmap.height / 4 * 4
        val argb = IntArray(inputWidth * inputHeight)

        //Log.i(TAG, "scaled : " + scaled);
        bitmap.getPixels(argb, 0, inputWidth, 0, 0, inputWidth, inputHeight)

        val yuv = ByteArray(inputWidth * inputHeight * 3 / 2)

        encodeYUV420SP(yuv, argb, inputWidth, inputHeight)

        return yuv
    }

    /**
     * rgba 转 yuv420 sp
     */
    private fun encodeYUV420SP(yuv420sp: ByteArray, argb: IntArray, width: Int, height: Int) {
        val frameSize = width * height
        var yIndex = 0
        var uvIndex = frameSize
        var a: Int
        var R: Int
        var G: Int
        var B: Int
        var Y: Int
        var U: Int
        var V: Int
        var index = 0
        for (j in 0 until height) {
            for (i in 0 until width) {
                a = argb[index] and -0x1000000 shr 24 // a is not used obviously
                R = argb[index] and 0xff0000 shr 16
                G = argb[index] and 0xff00 shr 8
                B = argb[index] and 0xff shr 0
                // well known RGB to YUV algorithm
                Y = (66 * R + 129 * G + 25 * B + 128 shr 8) + 16
                V = (-38 * R - 74 * G + 112 * B + 128 shr 8) + 128 // Previously U
                U = (112 * R - 94 * G - 18 * B + 128 shr 8) + 128 // Previously V
                yuv420sp[yIndex++] = (if (Y < 0) 0 else if (Y > 255) 255 else Y).toByte()
                if (j % 2 == 0 && index % 2 == 0) {
                    yuv420sp[uvIndex++] = (if (V < 0) 0 else if (V > 255) 255 else V).toByte()
                    yuv420sp[uvIndex++] = (if (U < 0) 0 else if (U > 255) 255 else U).toByte()
                }
                index++
            }
        }
    }

    /**
     * 从本地文件中读取并且解码图片生成bitmap
     */
    private fun readBitmap(path: String): Bitmap? {
        var fis: FileInputStream? = null
        var baos: ByteArrayOutputStream? = null
        try {
            fis = FileInputStream(path)
            baos = ByteArrayOutputStream()

            val fileBuffer = ByteArray(2048)
            var len: Int
            while (fis.read(fileBuffer).also { len = it } >= 0) {
                for (i in 0 until len) {
                    fileBuffer[i] = fileBuffer[i].xor(0xF)
                }
                baos.write(fileBuffer, 0, len)
            }
            baos.flush()
            val imageBuffer = baos.toByteArray()
            val bitmap = BitmapFactory.decodeByteArray(imageBuffer, 0, imageBuffer.size)
            Log.i("ErrorRecodeVideoUtils", "bitmap:$bitmap")
            return bitmap
        } catch (e: Exception) {

        } finally {
            try {
                fis?.close()
            } catch (e: Exception) {

            }
            try {
                baos?.close()
            } catch (e: Exception) {

            }
        }
        return null
    }
}
```