---
title: 初识AAC
date: 2017-07-20 13:38:26
categories: Android
tags: 
- AAC
---
<Excerpt in index | 首页摘要> 
> 初识AAC
> <!-- more -->
> <The rest of contents | 余下全文> 

##  前提  ##
最近在看FFMPEG的东西，发现很多内容由于没有相关基础知识支撑 导致看得人云里雾里的。

所以决定先好好学习学习基础理论知识。

之前的文章:

[RGB的学习](https://gloomyer.com/2017/07/14/study_rgb_01/)

[YUV的学习](https://gloomyer.com/2017/07/13/study_yuv_01/)

[PCM的学习](https://gloomyer.com/2017/07/18/study_pcm_01/)

[初识H264](https://gloomyer.com/2017/07/19/study_h264_01/)

##  什么是AAC  ##
简单的说

AAC就是音频采样数据的一种编码格式.

通过对AAC的解码，我们就可以的到音频的原始采样数据。

拿到采样数据，才可以播放声音。

##  AAc的简单认识  ##

AAC原始码流是由一个一个的ADTS frame组成的。他们的结构如下图所示。
<table>
    <tr>
		<td>...</td>
        <td>ADTS frame</td>
		<td>ADTS frame</td>
		<td>ADTS frame</td>
		<td>ADTS frame</td>
		<td>...</td>
    </tr>
</table>

其中每个ADTS frame之间通过syncword（同步字）进行分隔。同步字为0xFFF（二进制“111111111111”）。

AAC码流解析的步骤就是首先从码流中搜索0x0FFF，分离出ADTS frame；然后再分析ADTS frame的首部各个字段。

##  代码解析AAC信息  ##

看看看代码吧

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>


int getAdslFrame(unsigned char* buffer, int buf_size, unsigned char* data ,int* data_size){
    int size = 0;

    //如果这两个为空,说明异常了.
    if(buffer == NULL || data == NULL) return -1;
    
    for(;;){
        //buf_size　小于７不满足syncword的要求，异常
        if(buf_size < 7)
            return -1;
        //满足这个条件就说明这是一个 syncword
        if((buffer[0] == 0xff) && ((buffer[1] & 0xf0) == 0xf0)){
            //获取大小
            size |= ((buffer[3] & 0x03) << 11);
            size |= buffer[4] << 3;
            size |= (buffer[5] & 0xe0) >> 5;
            break;
        }
        --buf_size;
        ++buffer;
    }

    //需要修正指针位置
    if(buf_size < size) return 1;

    //拷贝内存指针
    memcpy(data, buffer, size);
    *data_size = size; //将大小赋值给data_size

    return 0;//正常返回
}

int main() {
    FILE *fp_log = fopen("../aac/log.txt", "wb+");//日志输出文件
    FILE *fp = fopen("../aac/source.aac", "rb+");//aac源文件

    unsigned char *aacframe = (unsigned char*)malloc(1024 * 5); //创建可能的一帧大小
    unsigned char *aacbuffer = (unsigned char*)malloc(1024 * 1024); //创建缓存

    printf("-----+- ADTS Frame Table -+------+\n");
    fprintf(fp_log, "-----+- ADTS Frame Table -+------+\n");
    printf(" NUM | Profile | Frequency| Size |\n");
    fprintf(fp_log, " NUM | Profile | Frequency| Size |\n");
    printf("-----+---------+----------+------+\n");
    fprintf(fp_log, "-----+---------+----------+------+\n");

    int offset = 0, size, cnt;
    while(!feof(fp)){
        int data_size = fread(aacbuffer + offset, 1, 1024 * 1024 - offset, fp);//尝试读取一个可能的缓存大小
        unsigned char* input_data = aacbuffer; //保存一下头指针

        for(;;){
            int ret = getAdslFrame(input_data, data_size, aacframe, &size); //读取一帧,返回读取成功还是失败

            if(ret == -1) break;//失败
            else if (ret == 1) {//修正指针位置
                memcpy(aacbuffer, input_data, data_size);
                offset = data_size;
                break;
            }
            
            //读取成功
            char profile_str[10] = {0};
            char frequence_str[10] = {0};
            
            unsigned char profile = aacframe[2] & 0xC0;
            profile = profile >> 6; 
            
            switch(profile){  
                case 0: sprintf(profile_str,"Main");break;
                case 1: sprintf(profile_str,"LC");break;
                case 2: sprintf(profile_str,"SSR");break;
                default:sprintf(profile_str,"unknown");break;
            }

            unsigned char sampling_frequency_index = aacframe[2] & 0x3C;
            sampling_frequency_index = sampling_frequency_index >> 2;
            
            //采样率
            switch(sampling_frequency_index){
                case 0: sprintf(frequence_str,"96000Hz");break;
                case 1: sprintf(frequence_str,"88200Hz");break;
                case 2: sprintf(frequence_str,"64000Hz");break;
                case 3: sprintf(frequence_str,"48000Hz");break;
                case 4: sprintf(frequence_str,"44100Hz");break;
                case 5: sprintf(frequence_str,"32000Hz");break;
                case 6: sprintf(frequence_str,"24000Hz");break;
                case 7: sprintf(frequence_str,"22050Hz");break;
                case 8: sprintf(frequence_str,"16000Hz");break;
                case 9: sprintf(frequence_str,"12000Hz");break;
                case 10: sprintf(frequence_str,"11025Hz");break;
                case 11: sprintf(frequence_str,"8000Hz");break;
                default:sprintf(frequence_str,"unknown");break;
            }

            //打印输出一帧的信息
            fprintf(fp_log, "%5d| %8s|  %8s| %5d|\n", cnt, profile_str, frequence_str, size);
            printf("%5d| %8s|  %8s| %5d|\n", cnt, profile_str, frequence_str, size);
            data_size -= size;
            input_data += size;
            cnt++;
        }
    }

    //释放资源
    fclose(fp);
    fclose(fp_log);
    free(aacbuffer);
    free(aacframe);
    return 0;
}
```

看看效果图

![](http://gloomyer.com/img/img/study_aac_01.png)

##  源码下载  ##

[Github打包下载](https://github.com/Gloomyer/study_aac_01)