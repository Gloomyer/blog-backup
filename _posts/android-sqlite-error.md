---
title: Android Sqlite 读取不到数据
date: 2016-9-27 11:32:11
categories: Android
tags: 
- Sqlite
---
<Excerpt in index | 首页摘要> 
>Android Sqlite 读取不到数据
<!-- more -->
<The rest of contents | 余下全文> 
  
今天，碰到一个相当奇葩的问题.  
  
在项目中，用户发表阶段，是需要从本地数据库中读取当前登录账户的uid,uidSid(安全验证)这两个字段的.  
  
原来的sql查询语句是:
```sql
select * from t_user;
```
然后跑项目的时候报NullPointException了.  
  
然后去看logcat，发现是在getUser()这个方法返回了null?  
  
怎么可能!  
  
百思不得其解,网上搜索也没有搜索到这方面的资料.  
  
最后是修改sql查询语句为:  
```sql
select * from t_user where 1 = 1;
```
才查到数据的.此坑，百思不得其解!