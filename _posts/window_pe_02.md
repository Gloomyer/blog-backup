---
title: Windows PE 结构学习2
date: 2019-03-05 15:35:32
categories: Hack
tags:
- PE
- Windows
---
<Excerpt in index | 首页摘要>
> Windows PE 结构学习
>
<!-- more -->
<The rest of contents | 余下全文>  

## 概要

![完整PE结构图](https://gloomyer.com/img/img/pe_struct.jpg)


今天主要讲解什么是节/数据目录/rva 是什么

### 节表和节
#### Windows如何将PE文件映射至内存
在执行/加载一个PE文件的时候,系统并不是开始就将整个文件并入内存,而是采用映射关系机制.PE装载器(系统)只建立号虚拟内存地址(RVA)和PE文件的映射关系.而只有当真正执行到某个内存页的指令或者访问某一页的数据时,这个页面才会从磁盘提交到物理内存.

系统装载可执行文件的方法又不完全等同于内存映射文件。当使用内存映射文件的时候，系统对“原著”相当忠实，如果将磁盘文件和内存映像比较的话，可以发现不管是数据本身还是数据之间的相对位置的都是完全相同的。而我们知道，在装载可执行文件的时候，有些数据在装入前会被预处理，如重定位等，正因此，装入以后，数据之间的相对位置可能发生微妙的变化。

操作系统还有内存隔离的特性,比如一个电脑是4GB内存,那么在每一个被装载进内存的程序看来.都是自己独享这4GB的内存.所以在EXE程序的执行,内存基址基本上总是装载在ImageBase(建议内存装载地址 vc一般为40000).dll 因为一个程序可能加载N个dll 所以dll 的建议装载地址未必生效.

#### 文件镜像<->内存镜像加载对照图
![文件镜像<->内存镜像加载对照图](https://gloomyer.com/img/img/pe_load_table.png)

### 偏移算法
节的起始地址在磁盘文件中是按照 IMAGE_OPTIONAL_HEADER 结构的 FileAlignment 字段的值进行对齐的，而当被加载到内存中时是按照同一结构中的 SectionAlignment 字段的值对其的，两者的值可能不同，所以一个节被装入内存后相对于文件头的偏移和在磁盘文件中的偏移可能是不同的。

但是,比如有一个数据在.text节中,.text的在文件镜像中的基址是0x50000 数据的偏移是0x5 那么它在文件镜像中的完整地址就是0x50005 , 在PE文件被载入内存之后 根据对其属性 .txt的基址是0x70000 那么它的偏移是不会发生改变的.那么它在内存镜像中的完整地址就是 0x70005

所以一个在节表中的数据地址RVA的算法就是 (数据在文件镜像中的真实地址-数据在文件镜像中的所在区段基址)+数据所在区段内存镜像基址


### 节表结构 IMAGE_SECTION_HEADER

#### 节表的开始与结束
节表在文件镜像中是紧跟在IMAGE_OPTIONAL_HEADER之后

具体个数不定!

由PE文件头 IMAGE_NT_HEADERS 结构中的 FileHeader.NumberOfSections 字段来指定的

#### 节表的结构
IMAGE_SECTION_HEADER
```
typedef struct _IMAGE_SECTION_HEADER {
  BYTE  Name[IMAGE_SIZEOF_SHORT_NAME];
  union {
    DWORD PhysicalAddress;
    DWORD VirtualSize;
  } Misc;
  DWORD VirtualAddress;
  DWORD SizeOfRawData;
  DWORD PointerToRawData;
  DWORD PointerToRelocations;
  DWORD PointerToLinenumbers;
  WORD  NumberOfRelocations;
  WORD  NumberOfLinenumbers;
  DWORD Characteristics;
} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
```

name:
>多数区块名都习惯性以一个“.”作为开头（例如：.text），这个“.” 实际上是不是必须的。值得我们注意的是，如果区块名达到8 个字节，后面就没有0字符了。并且前边带有一个“$” 的区块名字会从连接器那里得到特殊的待遇，前边带有“$” 的相同名字的区块在载入时候将会被合并，在合并之后的区块中，他们是按照“$” 后边的字符的字母顺序进行合并的。每个区块的名称都是唯一的，不能有同名的两个区块。但事实上节的名称不代表任何含义，他的存在仅仅是为了正规统一编程的时候方便程序员查看方便而设置的一个标记而已。所以将包含代码的区块命名为“.Data” 或者说将包含数据的区块命名为“.Code” 都是合法的。
当我们要从PE 文件中读取需要的区块时候，不能以区块的名称作为定位的标准和依据，正确的方法是按照 IMAGE_OPTIONAL_HEADER 结构中的数据目录字段结合进行定位

Misc:
>union 基本采用 VirtualSize 对表对应的区块的大小，这是区块的数据在没有进行对齐处理前的实际大小

VirtualAddress:
>该区块装载到内存中的RVA 地址。这个地址是按照内存页来对齐的，因此它的数值总是 SectionAlignment 的值的整数倍。

PointerToRawData:
>指出节在磁盘文件中所处的位置。这个数值是从文件头开始算起的偏移量 可以认为是这个区块在文件镜像中的基址

SizeOfRawData:
>该区块在磁盘中所占的大小，这个数值等于VirtualSize字段的值按照FileAlignment的值对齐以后的大小

Characteristics:
>该区块的属性。该字段是按位来指出区块的属性（如代码/数据/可读/可写等）的标志

### 数据目录

#### IMAGE_DATA_DIRCTORY 定义
IMAGE_DATA_DIRCTORY
```
typedef struct _IMAGE_DATA_DIRECTORY {
	DWORD　VirtualAddress; //相对虚拟地址
	DWORD　Size;　　　　　 //大小
} IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;
```

#### 数据目录做什么用的?
顾名思义,数据目录就是PE文件中存储真正数据的地方.固定有16个
比如图标/菜单/图片资源等等...
```
#define IMAGE_DIRECTORY_ENTRY_EXPORT		 0 导出表
#define IMAGE_DIRECTORY_ENTRY_IMPORT		 1 导入表 
#define IMAGE_DIRECTORY_ENTRY_RESOURCE		 2 资源目录
#define IMAGE_DIRECTORY_ENTRY_EXCEPTION		 3 异常目录
#define IMAGE_DIRECTORY_ENTRY_SECURITY		 4 安全目录
#define IMAGE_DIRECTORY_ENTRY_BASERELOC	         5 重定位基本表
#define IMAGE_DIRECTORY_ENTRY_DEBUG		 6 调试目录
#define IMAGE_DIRECTORY_ENTRY_COPYRIGHT		 7 描术字串
#define IMAGE_DIRECTORY_ENTRY_GLOBALPTR		 8 机器值
#define IMAGE_DIRECTORY_ENTRY_TLS		 9 TLS目录
#define IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG	 10 载入配值目录
#define IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT       11 绑定输入表
#define IMAGE_DIRECTORY_ENTRY_IAT		 12 导入地址表
#define IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT	 13 延迟载入描述
#define IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR     14 COM信息
```
其实一共有16个但是第十六个保留不适用所以没有写