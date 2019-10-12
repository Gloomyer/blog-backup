---
title: Windows PE 结构学习1
date: 2019-02-28 15:35:32
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
PE文件的全称是Portable Executable，意为可移植的可执行的文件，常见的EXE、DLL、OCX、SYS、COM都是PE文件，PE文件是微软Windows操作系统上的程序文件（可能是间接被执行，如DLL）

简单的说，所有能在Windows上被直接执行或者间接加载执行的就是属于PE文件 比如exe、dll

PE文件是一个非常复杂的结构

有图有真相

![完整PE结构图](https://gloomyer.com/img/img/pe_struct.jpg)

一个操作系统的可执行文件格式在很多方面是这个系统的一面镜子。虽然学习一个可执行文件格式通常不是一个程序员的首要任务，但是你可以从这其中学到大量的知识。

<font size='4' color='red'>阅读需要了解一定c/c++语法,了解大小端存储</font>

### DOS头
每一个PE文件都是以DOS头结构开始的，dos头是winDos时代就有了，之所以现在依然有，就是为了兼容dos时代的程序。

PE头所有的结构体都在winnt.h中

定义如下
```
typedef struct _IMAGE_DOS_HEADER {      // DOS .EXE header
    WORD   e_magic;                     // Magic number
    WORD   e_cblp;                      // Bytes on last page of file
    WORD   e_cp;                        // Pages in file
    WORD   e_crlc;                      // Relocations
    WORD   e_cparhdr;                   // Size of header in paragraphs
    WORD   e_minalloc;                  // Minimum extra paragraphs needed
    WORD   e_maxalloc;                  // Maximum extra paragraphs needed
    WORD   e_ss;                        // Initial (relative) SS value
    WORD   e_sp;                        // Initial SP value
    WORD   e_csum;                      // Checksum
    WORD   e_ip;                        // Initial IP value
    WORD   e_cs;                        // Initial (relative) CS value
    WORD   e_lfarlc;                    // File address of relocation table
    WORD   e_ovno;                      // Overlay number
    WORD   e_res[4];                    // Reserved words
    WORD   e_oemid;                     // OEM identifier (for e_oeminfo)
    WORD   e_oeminfo;                   // OEM information; e_oemid specific
    WORD   e_res2[10];                  // Reserved words
    LONG   e_lfanew;                    // File address of new exe header
  } IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
```

这里面我们常用的数据有两个
1. e_magic 
2. e_lfanew

第一个是一个表示，恒定为[5A,4D] ansi为MZ 　一般用于判断是否是DOS_HEADER

第二个表示NT_HEADER的偏移地址 (计算方法　０+e_lfanew) 0代表基址

### NT_HEADER

NTHeader 在winnt.h中不存在，　他的结构如下
NTHeader的结构如下:
```
struct NT_HEADER {
    DWORD signature; //ansi字符串　恒定为50 45 PE
    struct IMAGE_FILE_HEADER fileHeader; //后续介绍
    struct IMAGE_OPTIONAL_HEADER optionalHeader;//后续介绍
}
```

### IMAGE_FILE_HEADER

定义
```
typedef struct _IMAGE_FILE_HEADER {
  WORD  Machine;
  WORD  NumberOfSections;
  DWORD TimeDateStamp;
  DWORD PointerToSymbolTable;
  DWORD NumberOfSymbols;
  WORD  SizeOfOptionalHeader;
  WORD  Characteristics;
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;
```

这里面就有一些我们需要的数据了,我这里只介绍一些基本能用到的，用不到的不介绍了，可以自行查询。

Machine:
>这个PE文件的类型,只会为三个值0x014C(x86架构程序) 0x0200(英特尔安腾架构　已基本废弃) 0x8664(64架构程序)

NumberOfSections:
>表示节(块、区段)的数量（节、块、区段　翻译的区别，后续介绍）

TimeDateStamp:
>文件编译的时间

SizeOfOptionalHeader:
>IMAGE_OPTIONAL_HEADER　的大小(后续介绍)

Characteristics:
>文件属性 值含义很多，具体参照[官方文档](https://docs.microsoft.com/zh-cn/windows/desktop/api/winnt/ns-winnt-_image_file_header)


### IMAGE_OPTIONAL_HEADER

定义:
```
typedef struct _IMAGE_OPTIONAL_HEADER {
    //
    // Standard fields.
    //

    WORD    Magic;
    BYTE    MajorLinkerVersion;
    BYTE    MinorLinkerVersion;
    DWORD   SizeOfCode;
    DWORD   SizeOfInitializedData;
    DWORD   SizeOfUninitializedData;
    DWORD   AddressOfEntryPoint;
    DWORD   BaseOfCode;
    DWORD   BaseOfData;

    //
    // NT additional fields.
    //

    DWORD   ImageBase;
    DWORD   SectionAlignment;
    DWORD   FileAlignment;
    WORD    MajorOperatingSystemVersion;
    WORD    MinorOperatingSystemVersion;
    WORD    MajorImageVersion;
    WORD    MinorImageVersion;
    WORD    MajorSubsystemVersion;
    WORD    MinorSubsystemVersion;
    DWORD   Win32VersionValue;
    DWORD   SizeOfImage;
    DWORD   SizeOfHeaders;
    DWORD   CheckSum;
    WORD    Subsystem;
    WORD    DllCharacteristics;
    DWORD   SizeOfStackReserve;
    DWORD   SizeOfStackCommit;
    DWORD   SizeOfHeapReserve;
    DWORD   SizeOfHeapCommit;
    DWORD   LoaderFlags;
    DWORD   NumberOfRvaAndSizes;
    IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
} IMAGE_OPTIONAL_HEADER32, *PIMAGE_OPTIONAL_HEADER32;
```

看着很多，有用的的数据并不多

AddressOfEntryPoint:
>程序执行入口RVA (RVA 相对虚拟地址，后续介绍)

ImageBase:
>程序首选装入地址，一般为0x40000 可以在编译的时候修改，但是这个是建议地址，具体看执行的时候操作系统如何分配

SectionAlignment:
>内存区块对齐大小

Subsystem:
>当前程序是针对什么平台开发的 [具体取值参考地址](https://docs.microsoft.com/zh-cn/windows/desktop/api/winnt/ns-winnt-image_optional_header32)

FileAlignment:
>文件区块对齐大小

NumberOfRvaAndSizes:
>数据目录的数组数量，当下基本恒定为16

IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES]
>数据目录数组　基本恒定16个 具体数据的存放空间