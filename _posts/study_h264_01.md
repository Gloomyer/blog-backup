---
title: 初识H264
date: 2017-7-19 14:44:02
categories: Android
tags: 
- H264
---
<Excerpt in index | 首页摘要> 
> 初识H264
> <!-- more -->
> <The rest of contents | 余下全文> 

##  前提  ##
最近在看FFMPEG的东西，发现很多内容由于没有相关基础知识支撑 导致看得人云里雾里的。

所以决定先好好学习学习基础理论知识。

之前的文章:

[RGB的学习](https://gloomyer.com/2017/07/14/study_rgb_01/)

[YUV的学习](https://gloomyer.com/2017/07/13/study_yuv_01/)

[PCM的学习](https://gloomyer.com/2017/07/18/study_pcm_01/)

##  什么是H264  ##
简单的说

H264就是视频采样数据的一种编码格式.

通过对H264的解码，我们就可以的到视频的原始采样数据。

拿到采样数据，才可以播放视频。

##  H264的简单认识  ##

H.264原始码流（又称为“裸流”）是由一个一个的NALU组成的。

<table>
    <tr>
		<td>...</td>
        <td>NALU</td>
		<td>NALU</td>
		<td>NALU</td>
		<td>NALU</td>
		<td>...</td>
    </tr>
</table>

其中每个NALU之间通过startcode（起始码）进行分隔，起始码分成两种：0x000001（3Byte）或者0x00000001（4Byte）。

如果NALU对应的Slice为一帧的开始就用0x00000001，否则就用0x000001。

H.264码流解析的步骤就是首先从码流中搜索0x000001和0x00000001，分离出NALU；然后再分析NALU的各个字段。

##  代码解析H264Nalu信息  ##

直接看代码吧

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <windows.h>

typedef enum {  
		NALU_TYPE_SLICE    = 1,
		NALU_TYPE_DPA      = 2,
		NALU_TYPE_DPB      = 3,
		NALU_TYPE_DPC      = 4,
		NALU_TYPE_IDR      = 5,
		NALU_TYPE_SEI      = 6,
		NALU_TYPE_SPS      = 7,
		NALU_TYPE_PPS      = 8,
		NALU_TYPE_AUD      = 9,
		NALU_TYPE_EOSEQ    = 10,
		NALU_TYPE_EOSTREAM = 11,
		NALU_TYPE_FILL     = 12,
} NaluType;

typedef enum {  
    NALU_PRIORITY_DISPOSABLE = 0,
	NALU_PRIRITY_LOW         = 1,
	NALU_PRIORITY_HIGH       = 2,
	NALU_PRIORITY_HIGHEST    = 3
} NaluPriority;

typedef struct{		
    int startcodeprefix_len;
    unsigned len;
    unsigned max_size;
    int forbidden_bit;
    int nal_reference_idc;
    int nal_unit_type;
    char *buf;
} NALU_t;

int findStartCode2(unsigned char *buffer){  
    if(buffer[0]!=0 || buffer[1]!=0 || buffer[2] !=1) return 0; //0x000001?
    else return 1;
}

int findStartCode3 (unsigned char *buffer){  
    if(buffer[0]!=0 || buffer[1]!=0 || buffer[2] !=0 || buffer[3] !=1) return 0;//0x00000001?  
    else return 1;  
} 

int getNextNALU(FILE *fp_h264, NALU_t *pNalu){
    int startCodeFound = 0, rewind;  
    unsigned char *buffer = (unsigned char*)malloc(100000 * sizeof(char));

	//首先假设，是3字节版本 设置内容为3字节版本
	int pos = 3;
	pNalu->startcodeprefix_len = 3;
	//如果读取的不够3个字节那么就说明不是一个完整的nalu,不继续了
	if (3 != fread(buffer, 1, 3, fp_h264)){
        free(buffer);
        return 0;
    }
	
	//按照三字节去尝试读取,如果是3字节版本返回0，否则返回1
	int info2 = findStartCode2(buffer), info3;
	if(info2 != 1){//等于1说明不是3字节版本
		//因为不是3字节版本那么前3个字节肯定是都是0了，直接去读取第四个字节，如果是1那么就说明就是四字节版本，如果不是那说明不是一个完整的nalu,就退出吧
		if(1 != fread(buffer + 3, 1, 1, fp_h264)){  
            free(buffer);  
            return 0;  
        }
		//记录类型
		pos = 4;
		pNalu->startcodeprefix_len = 4;
	}
	
	
	while (!startCodeFound){
		if(feof(fp_h264)){
			//如果是文件的末尾
			pNalu->len = (pos - 1) - pNalu->startcodeprefix_len;
			memcpy(pNalu->buf, &buffer[pNalu->startcodeprefix_len], pNalu->len);
			pNalu->forbidden_bit = pNalu->buf[0] & 0x80;
			pNalu->nal_reference_idc = pNalu->buf[0] & 0x60;
			pNalu->nal_unit_type = pNalu->buf[0] & 0x1f;
			free(buffer);
			return pos - 1;
		}
		
		buffer[pos++] = fgetc(fp_h264);
		info3 = findStartCode3(&buffer[pos - 4]);
		if (info3 != 1){
			info2 = findStartCode2(&buffer[pos - 3]);
		}
		startCodeFound = (info2 == 1 || info3 == 1);
	}
	
	rewind = (info3 == 1) ? -4 : -3;
	
	if(0 != fseek(fp_h264, rewind, SEEK_CUR))
		free(buffer);

	pNalu->len = (pos + rewind) - pNalu->startcodeprefix_len;
	memcpy(pNalu->buf, &buffer[pNalu->startcodeprefix_len], pNalu->len);
	pNalu->forbidden_bit = pNalu->buf[0] & 0x80;
	pNalu->nal_reference_idc = pNalu->buf[0] & 0x60;
	pNalu->nal_unit_type = pNalu->buf[0] & 0x1f;

	free(buffer);
	return pos + rewind;
}

int main(){
	//打开H264文件
	const char * url = "../video/source.h264";
	FILE *fp_h264 = fopen(url, "rb+");
	FILE *fp_put = fopen("../video/output_log.log","wb+");
	
	//动态申请内存，设置可能存在的最大值
	NALU_t *pNalu = (NALU_t*)malloc(sizeof(NALU_t));
	pNalu->max_size = 100000;
	pNalu->buf = (char*)malloc(100000 * sizeof (char));

	//尝试读取内容,并且打印输出
	int data_offset = 0, nalu_num = 0;
	printf("-----+-------- NALU Table ------+---------+\n");
	fprintf(fp_put, "-----+-------- NALU Table ------+---------+\n");
    printf(" NUM |    POS  |    IDC |  TYPE |   LEN   |\n");
	fprintf(fp_put, " NUM |    POS  |    IDC |  TYPE |   LEN   |\n");
    printf("-----+---------+--------+-------+---------+\n");
	fprintf(fp_put, "-----+---------+--------+-------+---------+\n");

	//循环读取内容
	while(!feof(fp_h264)){
		int data_lenth;
		data_lenth = getNextNALU(fp_h264, pNalu);//循环获取一个nalu

		//判断类型
		char type_str[20] = {0};
		
		switch(pNalu->nal_unit_type){
			case NALU_TYPE_SLICE:sprintf(type_str,"SLICE");break;
			case NALU_TYPE_DPA:sprintf(type_str,"DPA");break;
            case NALU_TYPE_DPB:sprintf(type_str,"DPB");break;
            case NALU_TYPE_DPC:sprintf(type_str,"DPC");break;
            case NALU_TYPE_IDR:sprintf(type_str,"IDR");break;
            case NALU_TYPE_SEI:sprintf(type_str,"SEI");break;
            case NALU_TYPE_SPS:sprintf(type_str,"SPS");break;
            case NALU_TYPE_PPS:sprintf(type_str,"PPS");break;
            case NALU_TYPE_AUD:sprintf(type_str,"AUD");break;
            case NALU_TYPE_EOSEQ:sprintf(type_str,"EOSEQ");break;
            case NALU_TYPE_EOSTREAM:sprintf(type_str,"EOSTREAM");break;
            case NALU_TYPE_FILL:sprintf(type_str,"FILL");break;
		}
		
		char idc_str[20]={0};  
        switch(pNalu->nal_reference_idc >> 5){  
			case NALU_PRIORITY_DISPOSABLE:sprintf(idc_str,"DISPOS");break;  
			case NALU_PRIRITY_LOW:sprintf(idc_str,"LOW");break;  
			case NALU_PRIORITY_HIGH:sprintf(idc_str,"HIGH");break;  
			case NALU_PRIORITY_HIGHEST:sprintf(idc_str,"HIGHEST");break;  
        }
		
		//打印数据到屏幕和log文件
		fprintf(fp_put,"%5d| %8d| %7s| %6s| %8d|\n", nalu_num, data_offset, idc_str, type_str, pNalu->len);
		printf("%5d| %8d| %7s| %6s| %8d|\n", nalu_num, data_offset, idc_str, type_str, pNalu->len);

		data_offset = data_offset + data_lenth;
		//Sleep(800);
		nalu_num++;
	}

	//释放资源
	if (pNalu){
        if (pNalu->buf){
            free(pNalu->buf);
            pNalu->buf = NULL;
        }
        free(pNalu);
    }
	fclose(fp_h264);
	return 0;
}
```

注释写的比较全了。看看执行后的效果图:

![](http://gloomyer.com/img/img/study_h264_01.png)

##  源码下载  ##

[Github打包下载](https://github.com/Gloomyer/study_h264_01)