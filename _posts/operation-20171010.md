---
title: 服务器运维 nginx 403 ss搭建
date: 2017-10-10 13:40:10
categories: JavaWeb
---
<Excerpt in index | 首页摘要> 
> 服务器运维 nginx 403 ss搭建 
<!-- more -->
<The rest of contents | 余下全文> 

## shadowsocks ##

最近Number Big会议召开,很多翻墙软件都炸了。

我买的lantern专业版也是如此。

无奈之下只好打算自己动手丰衣足食了。

前提需要你有一台国外的服务器。

我这里采用的方式是搭建shadowsocks服务器。

shadowsocks是什么就不阐述了自己去百度吧。


一串的shell 按顺序执行即可。

```
yum update //更新本地的库

yum install gcc //安装gcc 根据情况看是否需要执行

yum install openssl-devel // open ssl 看情况是否需要执行

yum -y install wget //wget 下载工具 看情况是否需要执行


//安装python2.7 执行之前先运行python 看是否已经有了
wget http://python.org/ftp/python/2.7.3/Python-2.7.3.tar.bz2
tar -jxvf Python-2.7.3.tar.bz2 
cd Python-2.7.3  
./configure
make all
make install 
mv /usr/bin/python /usr/bin/python2.6.//
//指定center的版本
vi /usr/bin/yum  
//将文件头部的 
#!/usr/bin/python
//更换为
#!/usr/bin/python2.6.6
//以上是没有安装python2.7的步骤 如果安装了就不用管了


//安装pip 看情况是否需要执行
yum install python-setuptools
wget https://bootstrap.pypa.io/ez_setup.py -O - | python
wget http://pypi.python.org/packages/source/d/distribute/distribute-0.6.10.tar.gz
tar zxvf distribute-0.6.10.tar.gz
cd distribute-0.6.10
python setup.py install
cd
easy_install pip

//重点，这句话就是安装shadowsocks的服务
pip install shadowsocks
```

至此，我们的shadowsocks安装就完毕了。然后配置环境和启动即可。

配置环境
```
 vi /etc/shadowsocks.json
 
//然后输入以下内容:
//注意，第一行域名 IP都可以。
{
"server":"gloomyer.com", 
"port_password":{
     "6666":"password1",
     "7777":"password2"
     },
"timeout":60,
"method":"aes-256-cfb",
"fast_open":false,
"workers":1
}
```

启动shadowsocks服务
```
ssserver -c /etc/shadowsocks.json -d start
```
关闭shadowsocks服务
```
ssserver -c /etc/shadowsocks.json -d stop
```
重启shadowsocks服务
```
ssserver -c /etc/shadowsocks.json -d restart
```

配置开机自动启动
```
vi /etc/rc.local

ssserver -c /etc/shadowsocks.json -d start
```

## nginx 403 ##

我这里降ss完美跑起来之后发现我的博客挂了.

一直显示403.

一般nginx显示403有2个原因。

1，你的根目录下没有指定的首页文件，比如你配置的首页是index.html index.htm 但是你的根目录下并没有这个文件就可能会导致403

2，权限问题。你的网站除了网站本身权限 包括这个网站的上级所有目录权限都必须为775，切不能为/root下

第二点是我搜索到的的，但是我的博客之前是就是在/root下。权限也都分配了。之前是正常的，不知道为什么跑了ss之后就不行了。

解决方式就是2点，先将/root和网站的目录设置775权限，其次打开nginx的配置文件，

在第一行加入：

```
user    root;//如果有一个user 就把那个user注释或者后面替换为root.
```
