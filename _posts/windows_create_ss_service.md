---
title: Windows服务器1分钟搭建SS服务器
date: 2018-6-12 20:44:54
categories: 计算机
tags: 
- 计算机
- ss服务器
---
<Excerpt in index | 首页摘要> 
>  Windows服务器1分钟搭建SS服务器
> 
> <!-- more -->
> <The rest of contents | 余下全文> 

买了一台香港的Windows服务器

利用的是[libQtShadowsocks项目](https://github.com/shadowsocks/libQtShadowsocks)

先去下载[下载地址](https://github.com/shadowsocks/libQtShadowsocks/releases) 

当前最新的是:shadowsocks-libqss-v2.0.2-win64.7z

解压出来,有一个shadowsocks-libqss.exe

写一个json

```
{
    "server":"::",
    "server_port":8388,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"Password",
    "timeout":600,
    "method":"aes-256-cfb",
    "http_proxy": false
}
```

::代表监听ip4+ip6 纯ip4这里填写127.0.0.1 就好

然后保存为config.json和shadowsocks-libqss.exe在同一个目录

然后写一个.bat(.cmd)

```
shadowsocks-libqss.exe -c config.json -S
```

然后打开这个bat即可，关闭dos窗口就是关闭服务器。就是这么easy.