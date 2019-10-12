---
title: Windows10安装MySql5.7
date: 2016-7-21 09:28:44
categories: JavaWeb
tags:
- Mysql5.7
- Windows10
---
<Excerpt in index | 首页摘要> 
> Windows10安装MySql5.7 
>
<!-- more -->
<The rest of contents | 余下全文>  
  
刚入手了新电脑，上个图先.  
  
![](http://www.gloomyer.com/img/工作台.png)  
  
刚入手电脑要干什么呢？肯定是装一堆乱七八糟的环境了，不得不说一下吊炸天的vistual studio2015， 活生生装了10个小时~~  
  
好了废话不扯，直接说正题。  
  
---
  
MySql从5.5(不确定)开始取消了msi安装配置模式安装MySql。  
  
Windows下也得和Linux下一样手动配置MySql文件了  
  
首先下载MySql，这个我就不说了 我是从甲骨文官网下载的，他需要你注册一个账户(方便给你发广告用的)  
  
我为了速度着想，决定把MySql安装到固态上，我的固态直接全部分给了系统盘，即C盘。  
  
首先将下载好的文件解压到D盘，然后复制进入C\Program Files\目录下。  
  
然后修改安全选项-》Everyone用户-》全部控制(怎么修改文件夹权限自己去百度吧，这个太简单了就不说了)  
  
然后将文件夹中的my-default.int 复制一份，取名为my.ini  
  
然后修改其中内容为如下:  
```shell
	[mysqld]
	#Mysql5.7修改了在字符集控制指令
	character-set-server = utf8
	#如果是服务器用的话，建议设大点。
	innodb_buffer_pool_size = 128M
	#基路径 MySql的安装路径
	basedir = C:/Program Files/MySql5.7
	#数据路径
	datadir = C:/Program Files/MySql5.7/data
	#监听端口
	port = 3306
	#务必要配置这个！
	skip-grant-tables  
```
Mysql5.7取消了第一次用户的免密登录，所以配置文件中必须加入如上代码  
  
然后将MySql安装目录/bin目录配置进入系统环境变量Path中。  
  
然后打开管理员权限的命令提示符  
  
依次输入 
```shell
	mysqld -install  
	mysqld --initialize  
	net start mysql

	//Mysql登录设置初始密码
	mysql -uroot
	use mysql;
	update mysql.user set authentication_string=password('123qwe') where user='root' and Host = 'localhost';
	flush privileges;
	quit;

	//重新设置
	net stop mysql
	mysqld -remove
```
打开刚才我们配置的my.ini  
  
在最后一行 skip-grant-tables之前加上# 或者删除这行也可以  
  
然后继续在命令提示符中依次输入  
```shell
	mysqld -install
	net start mysql
	mysql -uroot -p123qwe
	//设置你的密码
	set password for 'root'@'localhost'=password('你的新密码');  
	flush privileges;   
```
现在你的MySql就配置并且启动完成了，可以去服务管理里面设置MySql这个服务的启动方式为自动,免得每次使用都要net start mysql  
  
<font color=red>着重说明下，MySql5.7在配置my.ini中修改了原来的字符集控制命令，所以我这里没有配置，如果需要配置，请去MySql官方文档中去查询修改后的字符集名称</font>