---
title: 博客的迁移
date: 2016-11-01 19:41:51
categories: Life
tags: 
- Hexo
- centOS

---
<Excerpt in index | 首页摘要> 
> 利用CentOS搭建Hexo环境
> <!-- more -->
> <The rest of contents | 余下全文> 

##  引言  ##

引言与正文无关，可以跳过。

之前的博客是在腾讯租的cvm,当时为了速度考虑，租的上海的线路，一个月80。

心酸的我博客搭起来没有20天，被封了，因为国内对域名空间的管控很严格，强制备案。

oh,right. 备案 照相 写材料 发出去，等审核。前前后后大概花了20天。搞完了。

我以为这就算完了，现实又一次啪啪的打响我的脸。

大概2个月后，我身份证所在的县政府给我打了个电话。

说要当地政府需要备案一下。寄给了我个表格让我填写一下。

好吧,填就填呗。



又过了一周，又给我打了个电话，这回就开始扯淡了，让我去县政府开会！(当然打电话给我的人语气，态度都很好)

我说我人在杭州，没办法过去，这时我就感觉这事估计没完。



就在周一，又给我打了个电话。

说现在自治州这边，所有的网站都需要挂靠党支部。

WTF？

党支部？

are you kidding me?

事实证明我还是太年轻了。

我说我肯定找不到的。

那个兄弟也是给我推荐了一个办法，让我找到现在住的社区去，

他们那里肯定是有党支部的，然后开个文件寄回给他们。



简单说明了下情况，那个哥们说，

如果找不到党支部挂靠，可能是需要关站的。



既然如此，我只有注销了现在的备案信息。



##  概述  ##

既然国内的cvm不能用了，就考虑买个国外的vps.

听了下欢喜的推荐，[搬瓦工](https://bandwagonhost.com/)貌似做的还不错。

看了下基础款的价位:2.99$,折合人民币20.23，比我原来的便宜了三分之2。

缺点就是系统是从Windows要迁移至Linux了，没有Windows简单(简单的博客不需要多高的性能)。

好吧。二话不说先买了一个。install new os。

看看支持的系统CentOS/Ubuntu/debian/fedora/suse

常见的基本都支持了。

虽然我自己一直用的Ubuntu，说句实话，一直对这个发行版欠缺好感。

想了想，干脆用centOS算了。

于是装了centOS7.0 64bit.



##   CentOS安装Hexo ##

因为没用过centOS，所以一些坑不太清楚。

我是root账户登录，如果你不是请在每个命令前加sudo!

首先安装git

```shell
yum install git-core
```

其次安装node(现在最新的版本是6.9)

```shell
wget https://nodejs.org/dist/v6.9.1/node-v6.9.1-linux-x64.tar.xz
xz -d node-v6.9.1-linux-x64.tar.xz
tar -xvf node-v6.9.1-linux-x64.tar
mv node-v6.9.1-linux-x64 /usr/local/node/ //我习惯放在这个目录下，你可以根据喜好来
```

然后配置环境变量,最好在文件末尾添加

```shell
vim ~/.bash_profile
NODE_HOME=/usr/local/node/bin //这个路径是根据上面移动的路径变化的
PATH=$PATH:$NODE_HOME
:wq//保存
source ~/.bash_profile //更新下缓存
```

安装hexo

```shell
npm install -g hexo
```

wow,right. 现在就是你自己配置你自己的博客了。

因为我有老的博客，所以还需拷贝到服务器上。利用scp命令即可。

这里记个坑

scp 后面指定服务器的端口 是-P不是-p。大小写区分!Fuck！！！

简单生成搞定之后，就是利用web服务运行网站了，我这里选择的是tomcat9.0

```shell
http://mirrors.hust.edu.cn/apache/tomcat/tomcat-9/v9.0.0.M11/bin/apache-tomcat-9.0.0.M11.tar.gz
tar -xzvf apache-tomcat-9.0.0.M11.tar.gz
mv apache-tomcat-9.0.0.M11.tar.gz /usr/local/tomcat
```

继续 编辑环境变量

```shell
vim ~/.bash_profile
NODE_HOME=/usr/local/node/bin //这个路径是根据上面移动的路径变化的
TOMCAT=/usr/local/tomcat/bin
PATH=$PATH:$NODE_HOME:$TOMCAT
:wq//保存
source ~/.bash_profile //更新下缓存
```

然后删除tamcat/webapp目录下所有文件/目录

然后就是发布hexo然后拷贝public到tomcat/webapp/目录下 更名为ROOT

去配置tomcat的启动端口

运行

```shell
startup.sh
```

好了，新博客到这里就已经搞好了!
