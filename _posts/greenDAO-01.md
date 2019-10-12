---
title: GreeDao-3.0 入门
date: 2017-01-11 17:25:27
categories: Android
tags:
- greenDao
---
<Excerpt in index | 首页摘要> 
> GreenDao 3.0
>
> 入门
>
> <!-- more -->
> <The rest of contents | 余下全文>  



### 概述 ###

大部分Android应用都会涉及到数据库的操作，而其中比较优秀的当属greendao。

greenDao是一款基于sqlite之上的轻量级的快速的ORM解决方案。



我自己的项目在重构开始，数据的存储需求并不是那么强烈。

所以当时直接选用了原生的sqlite纯手动的模式。



中途类似的需求慢慢增加。

考虑过使用greendao，那个时候grrendao是2.x的年代。各种繁琐的操作看得人心都累了。

自然就延误了下来。



如今稍微有点时间，打算正式切换数据库框架。后来发现了greendao到了3.x的年代。

官方也通过注解的方式帮助开发者更加快速的使用greendao。自然欣喜万分。



### 导入 ###

[GreenDao的Github](https://github.com/greenrobot/greenDAO)

首先，你需要在你的项目build.gradle文件中增加如下

```
dependencies {
  classpath 'org.greenrobot:greendao-gradle-plugin:3.0.0'
}
```

其次，到你的项目model(app)的build.gradle文件中增加如下

```
apply plugin: 'org.greenrobot.greendao'

dependencies {
  compile 'org.greenrobot:greendao:3.0.1'
}
```



然后在项目model(app)的build.gradle文件

加入如下标签

```
greendao {
    schemaVersion 6 //数据库的版本号
    targetGenDir 'src/main/java' //greenDAO辅助类的生成路径
    daoPackage 'com.qingxiang.ui.greendao' //greenDAO辅助类生成的包名
}
```



greenDAO在3.x中 让开发者通过注解帮助我们来生成原来繁琐的一些操作

列如：

```
/**
 * 搜索历史Bean
 */
@Entity
public class HistoryBean {
    private String value;
    private long time;

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    public long getTime() {
        return time;
    }

    public void setTime(long time) {
        this.time = time;
    }
}
```

@Entity注解告诉greenDAO这是一个要保存到数据库的实体类需要你帮我生成一个对应的操作文件



然后build，

你就会发现在你制定的目录下生成了

```
DaoMaster
DaoSession
HistoryBeanDao
```



然后我们新建一个dbmanager

```
package com.qingxiang.ui.engine;

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;

import com.qingxiang.ui.greendao.DaoMaster;

/**
 * GreenDao 数据库管理者
 */
public class DBManager {
    private final static String dbName = "db_qingxiang_greendao";
    private static DBManager mInstance;
    private Context mContext;
    private DaoMaster.DevOpenHelper openHelper;

    public DBManager(Context context) {
        this.mContext = context;
        openHelper = new DaoMaster.DevOpenHelper(context, dbName, null);
    }

    /**
     * 初始化,推荐在application中初始化
     *
     * @param myApp
     */
    public static void init(Context myApp) {
        mInstance = new DBManager(myApp);
    }

    /**
     * 获取单例引用
     *
     * @param
     * @return
     */
    public static DBManager getInstance() {
        return mInstance;
    }


    /**
     * 获取可读数据库
     */
    public SQLiteDatabase getReadableDatabase() {
        if (openHelper == null) {
            openHelper = new DaoMaster.DevOpenHelper(mContext, dbName, null);
        }
        SQLiteDatabase db = openHelper.getReadableDatabase();
        return db;
    }

    /**
     * 获取可写数据库
     */
    public SQLiteDatabase getWritableDatabase() {
        if (openHelper == null) {
            openHelper = new DaoMaster.DevOpenHelper(mContext, dbName, null);
        }
        SQLiteDatabase db = openHelper.getWritableDatabase();
        return db;
    }
}
```

然后我们来创建一个SearchHistoryDao

```
package com.qingxiang.ui.dao;

import com.qingxiang.ui.bean.HistoryBean;
import com.qingxiang.ui.engine.DBManager;
import com.qingxiang.ui.greendao.DaoMaster;
import com.qingxiang.ui.greendao.HistoryBeanDao;

import java.util.List;

/**
 * 搜索记录dao
 */
public class SearchHistoryDao {

    private DaoMaster daoMaster;

    private SearchHistoryDao() {
        daoMaster = new DaoMaster(DBManager.getInstance().getWritableDatabase());
    }

    private static SearchHistoryDao instance;

    public static SearchHistoryDao getInstance() {
        synchronized (SearchHistoryDao.class) {
            if (instance == null) {
                instance = new SearchHistoryDao();
            }
        }
        return instance;
    }

    /**
     * 获取所有的搜索记录
     */
    public List<HistoryBean> getHistory() {
        List<HistoryBean> list = daoMaster.newSession()
                .getHistoryBeanDao()
                .queryBuilder()
                .orderDesc(HistoryBeanDao.Properties.Time)
                .build()
                .list();
        return list;
    }

    /**
     * 增加或更新一条搜索记录
     *
     * @param value 增加或更新一条搜索记录
     */
    public void addHistory(String value) {
        daoMaster.newSession()
                .getHistoryBeanDao()
                .queryBuilder()
                .where(HistoryBeanDao.Properties.Value.eq(value))
                .buildDelete()
                .executeDeleteWithoutDetachingEntities();
        HistoryBean info = new HistoryBean(value, System.currentTimeMillis());
        daoMaster
                .newSession()
                .getHistoryBeanDao()
                .insert(info);
    }

    /**
     * 清空所有历史记录
     */
    public void clear() {
        daoMaster.newSession()
                .getHistoryBeanDao()
                .deleteAll();
    }

    /**
     * 删除某条历史记录
     *
     * @param value 要删除的历史记录
     */
    public void delete(String value) {
        daoMaster.newSession()
                .getHistoryBeanDao()
                .queryBuilder()
                .where(HistoryBeanDao.Properties.Value.eq(value))
                .buildDelete()
                .executeDeleteWithoutDetachingEntities();
    }
}
```



然后你就可以测试一下啦~



###  附录   ###

1.关于实体类的注解

​	@Entity 适用于类 告诉greenDAO这是一个要保存到数据库中的类

​		**schema**：告知GreenDao当前实体属于哪个schema

​		**active：**标记一个实体处于活动状态，活动实体有更新、删除和刷新方法

​		**nameInDb：**在数据中使用的别名，默认使用的是实体的类名

​		**indexes：**定义索引，可以跨越多个列

​		**createInDb：**标记创建数据库表

​	@Id 表明主键，需要为Long类型,可以通过@Id(autoincrement = true)设置自增长

​	@Property 用于设置属性在数据库中的列名（默认不写就是保持一致）@Property (nameInDb="newName")

​	@NotNull 非空

​	@Transient 标识这个字段是自定义的不会创建到数据库表里



2.build文件中的含义

- **schemaVersion**： 数据库schema版本，也可以理解为数据库版本号
- **daoPackage**：设置DaoMaster 、DaoSession、Dao包名
- **targetGenDir**：设置DaoMaster 、DaoSession、Dao目录
- **targetGenDirTest**：设置生成单元测试目录
- **generateTests**：设置自动生成单元测试用例