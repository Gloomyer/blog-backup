---
title: YUV的一些基础知识
date: 2017-7-13 02:52:19
categories: Android
tags: 
- yuv
---
<Excerpt in index | 首页摘要> 
> YUV的一些基础知识
> <!-- more -->
> <The rest of contents | 余下全文> 

##  前提  ##
最近在看FFMPEG的东西，发现很多内容由于没有相关基础知识支撑 导致看得人云里雾里的。

所以决定先好好学习学习基础理论知识。

##  什么是YUV  ##
之前做FFmepeg视频播放的时候，里面用到了一个工具类(livyuv)。

就是用于将ffmpeg读出来的每一个帧的视频(YUV)->rgb 然后显示出来.

那么这个YUV到底是什么呢?

[RGB和YUV](https://www.douban.com/note/76361504/):

```
RGB和YUV都是色彩空间，用于表示颜色，两者可以相互转化。
YUV（亦称YCrCb）是被欧洲电视系统所采用的一种颜色编码方法（属于PAL）。
YUV主要用于优化彩色视频信号的传输，使其向后兼容老式黑白电视。
与R GB视频信号传输相比，
它最大的优点在于只需占用极少的带宽（RGB要求三个独立的视频信号同时传输）
```

简单地说，YUV和RGB的作用是一样的，用来描述一个点的颜色。

RGB通过色彩来让人眼感知颜色，YUV通过亮度(这个很有趣，有兴趣的可以自己去了解一下)。

而且YUV比RGB更省空间！


##  YUV的处理  ##

现在大概搞懂了YUV是个什么东西。

YUV详尽的就自己去看一下吧 [YUV格式详解][1]
[1]: https://msdn.microsoft.com/en-us/library/aa904813(VS.80).aspx

#####  本文中像素的采样位数一律为8bit。由于1Byte=8bit，所以一个像素的一个分量的采样值占用1Byte。  ######

####  YUV420视频的分离YUV  ####

首先，你需要下载一个YUV视频播放器(文章结尾的打包下载中提供)

首先说一下，一个yuv文件的每一帧的大小(yuv420p)为

宽度 * 高度 * 3 / 2

其中，y占2/3的大小,u/v 各占1/6的大小.

那么首先看看我们的分离代码吧:

```
#include <stdio.h>
#include <stdlib.h>

int simplest_yuv420_split(int w, int h,int num){  
    FILE *fp = fopen("../video/yuv_420p.yuv", "rb+");
    FILE *fp1 = fopen("../video/output_420_y.y", "wb+");
    FILE *fp2 = fopen("../video/output_420_u.y", "wb+");
    FILE *fp3 = fopen("../video/output_420_v.y", "wb+");

    unsigned char * pic = (unsigned char *)malloc(w * h * 3 / 2);

    for(int i = 0; i < num; i++){
        fread(pic, 1, w * h * 3 / 2, fp);

        fwrite(pic, 1, w * h , fp1);
    
        fwrite(pic + w * h, 1, w * h / 4, fp2);

        fwrite(pic + w * h * 5 / 4, 1, w * h / 4, fp3);
    }

    fclose(fp3);
    fclose(fp2);
    fclose(fp1);
    fclose(fp);

    return 0;
}

int main(){
    simplest_yuv420_split(352, 288, 300); //这个300是说这个文件有300帧
    return 0;
}
```

跑起来之后让我们看一下对比图(顺序依次为:原图、Y、U、V):

在这里需要注意输出的U、V分量在YUV播放器中也是当做Y分量进行播放的

![](http://gloomyer.com/img/img/yuv/yuv_study_01.png)

![](http://gloomyer.com/img/img/yuv/yuv_study_02.png)

![](http://gloomyer.com/img/img/yuv/yuv_study_03.png)
![](http://gloomyer.com/img/img/yuv/yuv_study_04.png)

注意播放后三个视频的时候，像素格式选择为Y,然后分辨率大小需要调整(U/V)

这里的UV、文件大小是正常的大小/2,即原来的为352x288 u/v为176x144 


####  YUV444视频的分离YUV  ####

首先，你需要下载一个YUV视频播放器(文章结尾的打包下载中提供)

首先说一下，一个yuv文件的每一帧的大小(yuv444p)为

宽度 * 高度 * 3

其中，y/u/v各占1/3

那么首先看看我们的分离代码吧:

```
#include <stdio.h>
#include <stdlib.h>

int simplest_yuv444_split(int w, int h,int num){  
    FILE *fp = fopen("../video/yuv_444p.yuv", "rb+");
    FILE *fp1 = fopen("../video/output_444_y.y", "wb+");
    FILE *fp2 = fopen("../video/output_444_u.y", "wb+");
    FILE *fp3 = fopen("../video/output_444_v.y", "wb+");

    unsigned char * pic = (unsigned char *)malloc(w * h * 3);

    for(int i = 0; i < num; i++){
        fread(pic, 1, w * h * 3, fp);

        fwrite(pic, 1, w * h , fp1);
    
        fwrite(pic + w * h, 1, w * h, fp2);

        fwrite(pic + w * h * 2, 1, w * h, fp3);
    }

    fclose(fp3);
    fclose(fp2);
    fclose(fp1);
    fclose(fp);

    return 0;
}

int main(){
    simplest_yuv444_split(256, 256, 1);

    return 0;
}
```

跑起来之后让我们看一下对比图(顺序依次为:原图、Y、U、V):

在这里需要注意输出的U、V分量在YUV播放器中也是当做Y分量进行播放的

![](http://gloomyer.com/img/img/yuv/yuv_study_05.png)

![](http://gloomyer.com/img/img/yuv/yuv_study_06.png)

![](http://gloomyer.com/img/img/yuv/yuv_study_07.png)

![](http://gloomyer.com/img/img/yuv/yuv_study_08.png)

这里的视频文件大小都是256x256

播放的时候注意选择一下分辨率和像素格式


####  YUV420去掉像素颜色(灰度)  ####

先看代码

```
#include <stdio.h>
#include <stdlib.h>

int yuv420_gray(int w, int h,int num){  
    FILE *fp = fopen("../video/yuv_420p.yuv", "rb+");
    FILE *fp1 = fopen("../video/output_420_gray.yuv", "wb+");

    unsigned char * pic = (unsigned char *)malloc(w * h * 3 / 2);

    for(int i = 0; i < num; i++){
        fread(pic, 1, w * h * 3 / 2, fp);
		for(int j = 0; j < w * h / 2; j++){
			(pic + w * h)[j] = 128; //u/v修改为128
			//printf("%d  ", (pic + w * h)[j]);
		}
		fwrite(pic, 1, w * h * 3 / 2, fp1);
    }


    fclose(fp1);
    fclose(fp);

    return 0;
}

int main(){
    yuv420_gray(352, 288, 300);
    return 0;
}
```

从代码可以看出，如果想把YUV格式像素数据变成灰度图像，只需要将U、V分量设置成128即可。这是因为U、V是图像中的经过偏置处理的色度分量。色度分量在偏置处理前的取值范围是-128至127，这时候的无色对应的是“0”值。经过偏置后色度分量取值变成了0至255，因而此时的无色对应的就是128了

看一下效果:

![](http://gloomyer.com/img/img/yuv/yuv_study_09.png)

![](http://gloomyer.com/img/img/yuv/yuv_study_10.png)


####  YUV420亮度减半  ####

还是先看一下代码:

```
#include <stdio.h>
#include <stdlib.h>

int yuv420_halfy(int w, int h,int num){  
    FILE *fp = fopen("../video/yuv_420p.yuv", "rb+");
    FILE *fp1 = fopen("../video/output_420_half.yuv", "wb+");

    unsigned char * pic = (unsigned char *)malloc(w * h * 3 / 2);

    for(int i = 0; i < num; i++){
        fread(pic, 1, w * h * 3 / 2, fp);
		for(int j = 0; j < w * h; j++){
			pic[j] /= 2; //y减半
		}
		fwrite(pic, 1, w * h * 3 / 2, fp1);
    }


    fclose(fp1);
    fclose(fp);

    return 0;
}

int main(){
    yuv420_halfy(352, 288, 300);
    return 0;
}
```

很简单，就是将Y的值减半即可.

看一下效果图:

![](http://gloomyer.com/img/img/yuv/yuv_study_11.png)

![](http://gloomyer.com/img/img/yuv/yuv_study_12.png)

####  YUV420像素数据的周围加上边框  ####

就是将视频的周围N个像素点的亮度调整为255即可.

看一下代码:

```
#include <stdio.h>
#include <stdlib.h>

#define BORDER 10

int yuv420_border(int w, int h,int num){  
    FILE *fp = fopen("../video/yuv_420p.yuv", "rb+");
    FILE *fp1 = fopen("../video/output_420_border.yuv", "wb+");

    unsigned char * pic = (unsigned char *)malloc(w * h * 3 / 2);

    for(int i = 0; i < num; i++){
        fread(pic, 1, w * h * 3 / 2, fp);

		for(int j = 0;j < h; j++){
            for(int k = 0; k < w; k++){
				//这个判断就是为了计算出边框10个像素点的数据
                if(k < BORDER || k > (w-BORDER) || j < BORDER || j > ( h - BORDER)) 
                    pic[j*w+k]=255;
            }
        }

		fwrite(pic, 1, w * h * 3 / 2, fp1);
    }


    fclose(fp1);
    fclose(fp);

    return 0;
}

int main(){
    yuv420_border(352, 288, 300);
    return 0;
}
```

看一下效果图:

![](http://gloomyer.com/img/img/yuv/yuv_study_13.png)

![](http://gloomyer.com/img/img/yuv/yuv_study_14.png)


####  生成YUV420的灰阶图  ####

直接上代码:

```
#include <stdio.h>
#include <stdlib.h>

void create_gray_yuv420(int width, int height, int ymin, int ymax, int barnum){
	int barwidth;  
    float lum_inc;  
    unsigned char lum_temp;  
    int uv_width,uv_height;  
    FILE *fp=NULL;  
    unsigned char *data_y=NULL;  
    unsigned char *data_u=NULL;  
    unsigned char *data_v=NULL;  
    int t=0,i=0,j=0;

	barwidth=width/barnum;  
    lum_inc=((float)(ymax-ymin))/((float)(barnum-1));  
    uv_width=width/2;  
    uv_height=height/2;

	data_y=(unsigned char *)malloc(width*height);
    data_u=(unsigned char *)malloc(uv_width*uv_height);
    data_v=(unsigned char *)malloc(uv_width*uv_height);

	if((fp=fopen("../video/out_put_420p_graybar.yuv","wb+"))==NULL){  
        printf("Error: Cannot create file!");  
        return;  
    }

	printf("Y, U, V value from picture's left to right:\n");  
    for(t=0;t<(width/barwidth);t++){  
        lum_temp=ymin+(char)(t*lum_inc);  
        printf("%3d, 128, 128\n",lum_temp);  
    }

	for(j=0;j<height;j++){  
        for(i=0;i<width;i++){  
            t=i/barwidth;  
            lum_temp=ymin+(char)(t*lum_inc);  
            data_y[j*width+i]=lum_temp;  
        }  
    }

    for(j=0;j<uv_height;j++){  
        for(i=0;i<uv_width;i++){  
            data_u[j*uv_width+i]=128;  
        }  
    }

    for(j=0;j<uv_height;j++){  
        for(i=0;i<uv_width;i++){  
            data_v[j*uv_width+i]=128;  
        }  
    }
	
	fwrite(data_y,width*height,1,fp);
    fwrite(data_u,uv_width*uv_height,1,fp);
    fwrite(data_v,uv_width*uv_height,1,fp);
	fclose(fp);
    free(data_y);
    free(data_u);
    free(data_v);
}

int main(){
	create_gray_yuv420(352, 288,0,255,10);
	return 1;
}
```

这个就是根据宽度计算每个条目的宽度，然后U、V都是255， Y从0-255递增

看看效果图:

![](http://gloomyer.com/img/img/yuv/yuv_study_15.png)

####  源码下载  ####
ok，YUV的学习就到这里为止。

后面会继续讲讲RGB.

[Github下载](https://github.com/Gloomyer/yuv-study-01)