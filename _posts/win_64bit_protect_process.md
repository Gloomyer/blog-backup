---
title: Windows 64bit 驱动保护进程
date: 2018-11-12 21:23:32
categories: Hack
tags:
- 驱动开发
- Hook
---
<Excerpt in index | 首页摘要>
> Windows 64bit 驱动hook OpenProcess 保护进程
>
<!-- more -->
<The rest of contents | 余下全文>  

## 概要

Windows64Bit下驱动通过Hook NtOpenProcess实现对进程的保护

防止其他读写当前进程

## 例图
![GIF图](http://gloomyer.com/img/img/win_64bit_protect_process.gif)

## 代码
直接看Github [WinX64保护进程](https://github.com/Gloomyer/WinX64ProtectProcess)
