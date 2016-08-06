---
layout: post
title:  "Berkeley DB学习笔记"
categories: Database
tags: berkeleydb
---

* content
{:toc}

## 前言

> 多学个东西，即使只是个简单的demo，也当是开阔视野了。

其实如果换做是我，估计不太容易会引进这么个东东.....

不过既然接了别人留下来的业务，改学还是要学一下的，剩的以后出问题了，连这是个什么玩意都不知道。

## 简介

Berkeley DB是一个开源的文件数据库，介于关系数据库与内存数据库之间，使用方式与内存数据库类似，它提供的是一系列直接访问数据库的函数，而不是像关系数据库那样需要网络通讯、SQL解析等步骤。

一些属性：

- Berkeley DB Java Edition (JE)是一个完全用JAVA写的，它适合于管理海量的，简单的数据。

- 能够高效率的处理1到1百万条记录，制约JE数据库的往往是硬件系统，而不是JE本身。

- 多线程支持，JE使用超时的方式来处理线程间的死琐问题。

- Database都采用简单的key/value对应的形式。

- 高级的数据库特性，比如ACID 数据库事务处理，细粒度锁，XA接口，热备份以及同步复制。

## 示例

### 引入maven依赖

使用方式：maven。感觉使用maven挺方便的，没必要按照网上别人写的那样再下载文件解压配置路径什么的。

```
        <dependency>
            <groupId>com.sleepycat</groupId>
            <artifactId>je</artifactId>
            <version>4.0.92</version>
        </dependency>
```
###　初始化数据库信息

```
        Environment myDbEnvironment = null;
        Database myDatabase = null;

        EnvironmentConfig envConfig = new EnvironmentConfig();// 配置环境变量
        envConfig.setAllowCreate(true);
        File f = new File(dbEnv);
        if (!f.exists()) {
           f.mkdirs();
        }
        myDbEnvironment = new Environment(f, envConfig);

        DatabaseConfig dbConfig = new DatabaseConfig();// 打开数据库
        dbConfig.setAllowCreate(true);
        myDatabase = myDbEnvironment.openDatabase(null, "myDatabase", dbConfig);


```

### 插入数据

```
        String aKey = "key1";
        String aData = "data";
        DatabaseEntry theKey = new DatabaseEntry(aKey.getBytes("UTF-8"));
        DatabaseEntry theData = new DatabaseEntry(aData.getBytes("UTF-8"));
        myDatabase.put(null, theKey, theData);
```

### 读出数据

```
        DatabaseEntry theKey = new DatabaseEntry(aKey.getBytes("UTF-8"));
        DatabaseEntry resultData = new DatabaseEntry();
        OperationStatus res = myDatabase.get(null, theKey, resultData, LockMode.DEFAULT);
        if(res == OperationStatus.SUCCESS) {
            byte[] retData = resultData.getData();
            String foundData = new String(retData, "UTF-8");
            System.out.println(foundData); //
            System.out.println(myDatabase.count()); //读取有多少key/data对
            }
```

### 关闭连接


```
        if (myDatabase != null) {
            myDatabase.close();
        }
        if (myDbEnvironment != null) {
            myDbEnvironment.cleanLog(); //  在关闭环境前清理下日志
            myDbEnvironment.close();
        }
```

### **完整代码**

```
import com.sleepycat.je.*;

import java.io.File;
import java.io.UnsupportedEncodingException;

/**
 * Created by Dante on 2016/6/13.
 */
public class Test {
    private static String dbEnv = "D:\\workspace\\test\\bdb";

    public static void main(String[] args) {
        Environment myDbEnvironment = null;
        Database myDatabase = null;
        try {
            EnvironmentConfig envConfig = new EnvironmentConfig();// 配置环境变量
            envConfig.setAllowCreate(true);
            File f = new File(dbEnv);
            if (!f.exists()) {
                f.mkdirs();
            }
            myDbEnvironment = new Environment(f, envConfig);

        } catch (DatabaseException dbe) {
        }
        try {
            DatabaseConfig dbConfig = new DatabaseConfig();// 打开数据库
            dbConfig.setAllowCreate(true);
            myDatabase = myDbEnvironment.openDatabase(null, "myDatabase", dbConfig);

        } catch (DatabaseException dbe2) {

        }
        //存储数据
        String aKey = "key1";
        String aData = "data";
        try {
            DatabaseEntry theKey = new DatabaseEntry(aKey.getBytes("UTF-8"));
            DatabaseEntry theData = new DatabaseEntry(aData.getBytes("UTF-8"));
            myDatabase.put(null, theKey, theData);
        } catch (Exception e) {

        }

        //读取数据
        try {
            DatabaseEntry theKey = new DatabaseEntry(aKey.getBytes("UTF-8"));
            DatabaseEntry resultData = new DatabaseEntry();
            OperationStatus res = myDatabase.get(null, theKey, resultData, LockMode.DEFAULT);
            if(res == OperationStatus.SUCCESS) {
                byte[] retData = resultData.getData();
                String foundData = new String(retData, "UTF-8");
                System.out.println(myDatabase.count());
                System.out.println(foundData);
            }
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }

        // 关闭,应该会自动提交
        try {
            if (myDatabase != null) {
                myDatabase.close();
            }
            if (myDbEnvironment != null) {
                myDbEnvironment.cleanLog(); //  在关闭环境前清理下日志
                myDbEnvironment.close();
            }
        } catch (DatabaseException dbe) {

        }
    }


}
```

## 总结

虽说不太理解这个东东的原理，不过还是用着玩了一下。

***
2016-06-28 hzct
