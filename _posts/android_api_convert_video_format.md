---
title: android MediaMuxer+MediaExtractor 实现视频转格式
date: 2021-07-12 19:00:00
categories: Android
tags:
toc: true
---
>  通过 android api MediaMuxer&MediaExtractor 实现视频转格式
> <!-- more -->

## 概述

最近app内需要上传视频,为了保证视频播放在各端的统一,需要将视频封装格式转换为mp4

本来上ffmpeg非常简单,但是ffmpeg对包体的增加会比较大,而且需求功能很简单 就考虑用Android的media相关api来实现.

流程上来说就是 未知视频格式->解码->编码->mp4

经测试 非h265/h264 编码+mp4封装 ios支持 所以为了速度就不转码视频编码格式了

老视频中每读到一帧的数据 直接编码进新视频中就好了

## 代码

MediaExtractor 一次只能解析一个轨道,所以需要创建两个,一个负责视频解码,一个负责音频解码


```kotlin


import android.media.MediaCodec
import android.media.MediaExtractor
import android.media.MediaFormat
import android.media.MediaMuxer
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext
import shop.memall.newapp.base.CM
import java.io.File
import java.lang.Exception
import java.lang.RuntimeException
import java.nio.ByteBuffer
import java.util.concurrent.atomic.AtomicInteger

/**
 * 视频转码工具
 */
@Suppress("BlockingMethodInNonBlockingContext")
object VideoTranscodingUtils {

    /**
     * 转码输入视频为mp4
     */
    @JvmStatic
    suspend fun transcoding2Mp4(origin: String): String? {
        return withContext(Dispatchers.IO) {
            val dir = CM.mainApplication().getExternalFilesDir("transcodingVideos")
                ?: File(CM.mainApplication().cacheDir, "transcodingVideos")
            try {
                dir.mkdirs()
            } catch (ignore: Exception) {
            }
            val output = File(dir, "${System.currentTimeMillis()}.mp4")
            var mediaMuxer: MediaMuxer? = null
            try {
                val videoExtractor = MediaExtractor()
                val audioExtractor = MediaExtractor()
                mediaMuxer =
                    MediaMuxer(output.absolutePath, MediaMuxer.OutputFormat.MUXER_OUTPUT_MPEG_4)

                videoExtractor.setDataSource(origin)
                audioExtractor.setDataSource(origin)

                val numTracks: Int = videoExtractor.trackCount
                var muxerTrackNum = 0
                for (idx in 0 until numTracks) {
                    val trackFormat = videoExtractor.getTrackFormat(idx)
                    val mime = trackFormat.getString(MediaFormat.KEY_MIME) ?: ""
                    if (mime.contains("video")) {
                        videoExtractor.selectTrack(idx)
                        mediaMuxer.addTrack(trackFormat)
                        muxerTrackNum++
                    } else if (mime.contains("audio")) {
                        audioExtractor.selectTrack(idx)
                        mediaMuxer.addTrack(trackFormat)
                        muxerTrackNum++
                    }
                    if (muxerTrackNum >= 2) break
                    LG.fi("convert video track idx:$idx mime:$mime")
                }

                val atomicInteger = AtomicInteger(0)

                mediaMuxer.start()

                ThreadUtils.start(DecodeThread(videoExtractor, mediaMuxer, 0, atomicInteger))
                ThreadUtils.start(DecodeThread(audioExtractor, mediaMuxer, 1, atomicInteger))

                while (atomicInteger.get() < 2) {
                    Thread.sleep(0)
                }
                if (atomicInteger.get() > 2) throw RuntimeException("解码异常");
                output.absolutePath
            } catch (e: Throwable) {
                e.printStackTrace()
                output.delete()
                return@withContext null
            } finally {
                LG.fi("convert transcoding compile")
                mediaMuxer?.stop()
                mediaMuxer?.release()
            }
        }
    }

    /**
     * 解码线程
     */
    private class DecodeThread(
        val extractor: MediaExtractor,
        val mediaMuxer: MediaMuxer,
        val model: Int, //0:视频轨道,1:音频轨道
        val atomicInteger: AtomicInteger
    ) : Runnable {

        override fun run() {
            try {
                val buffer = ByteBuffer.allocate(500 * 1024)
                val bufferInfo = MediaCodec.BufferInfo()

                var firstTime = 0L
                var size: Int

                while (extractor.readSampleData(buffer, 0).also { size = it } > 0) {
                    val rackIndex = extractor.sampleTrackIndex
                    if (firstTime == 0L) firstTime = extractor.sampleTime
                    bufferInfo.presentationTimeUs = extractor.sampleTime - firstTime
                    bufferInfo.flags = extractor.sampleFlags
                    bufferInfo.size = size
                    mediaMuxer.writeSampleData(rackIndex, buffer, bufferInfo)
                    extractor.advance()
                }
                LG.fi("convert write success mode:$model")
                atomicInteger.addAndGet(1)
            } catch (e: Throwable) {
                e.printStackTrace()
                LG.fi("convert write has an exception mode:$model")
                atomicInteger.addAndGet(4)
            } finally {
                extractor.release()
            }
        }

    }
}

```