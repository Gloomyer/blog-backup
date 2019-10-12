---
title: MySql5.7开启root用户远程访问 
date: 2017-02-16 14:38:33
categories: JavaWeb
tags:
- MySql5.7
---
<Excerpt in index | 首页摘要> 
> MySql5.7开启root用户远程访问
>
> <!-- more -->
> <The rest of contents | 余下全文>  



服务器上装了mysql，

想在需要在本地用工具连接。

需要开启root用户的网络访问权限。



首先用服务器本地登录mysql.

```sql
mysql -uroot -pxxxxx
//切换至mysql的数据库
use mysql;
select user,host from user;
//我们能看到root 用户 后面跟的是localhost 如果需要指定具体的ip，那么就修改为具体的ip，如果需要开通所有就修改为%
update user set host =  '%' where user = 'root';
//修改完成退出mysql
```

然后找到myslq的配置文件 my.conf（一般在/etc/mysql/）



坑的地方来了，首先利用vim打开

原来熟悉的数据没有了，变成了

```shell
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
```

退出后，你查看着两个是两个目录.

没错。新版本的mysql将配置文件分离了！！！



我们要开启远程访问需要修改bind-address 这一条。

新的bind-address是在 mysql.conf.d目录下的mysqld.cnf文件里面

```shell
vim /etc/mysql/mysqld.conf.d/mysqld.cnf
//找到bind-address修改为
bind-address = 0.0.0.0
//保存，退出
service mysql restart //重启mysql
```

好了。现在就可以远程连你的服务器了。