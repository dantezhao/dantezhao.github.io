---
layout: post
title:  "Jdbc源码详解（一）：示例+Driver注册流程"
categories: 编程语言
tags:  sql jdbc java
---

* content
{:toc}


## 0x00 前言

自己一直说要某个开源项目的源码，但是一直没有真正地好好开始，一是以为看源码其实不容易看懂，而是因为选择犹豫，最后也敲定看哪个。

这次正式开始看jdbc的源码有两个三个：一是因为《java编程思想》这本书快看完了，折腾一个多月的时间，里面除了多线程和图形编程这两块基本都看得差不多了；一个是因为《设计模式之禅》这本书看了一半左右，里面的设计模式自己大致都明白是什么隐私，但是印象不深刻，需要有一个稳固的过程；一个是因为jdbc相对来说比较熟悉，加上以后肯定会常和数据库打交道，因此就决定看了。

在分析源码的时候，我会先自己理清思路，然后再回头来总结，总结的过程尽量地变成一个探索的过程，以此来梳理自己看源码的方法论。





## 0x01 阅读环境

- ide：idea15
- jdk：7
- mysql-connector-java：5.1.34
- h2：1.4.187

这次选了mysql和h2两款数据库的jdbc程序来分析，mysql是因为这个是最常用的，以后也会经常和它打交道，h2是因为它是java写的数据库，以后准备看它的源码，现在先提前了解一下。

## 0x02 jdbc示例

下面是一个最基本的jdbc示例，通过这个例子，后面我会详细地介绍整个流程。

### 第一个jdbc程序

这是一个最基本的jdbc连接程序，我省掉了异常处理。

```
public class JDBCTest {
    public static void main(String[] args) {
        Connection conn = null;
        Statement stmt = null;
        try{
            //STEP 1: Register JDBC driver
            Class.forName("com.mysql.jdbc.Driver");
            //STEP 2: Open a connection
            conn = DriverManager.getConnection("jdbc:mysql://192.168.108.145/test", "root", "root");
            //STEP 3: Execute a query
            stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery("select * from userinfo limit 1");
            //STEP 4: Get results
            while(rs.next()){
                System.out.println(rs.getString("id"));
            }
            rs.close();
        }catch(SQLException se){
            ......
        }//end try
    }
}
```
maven依赖
```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.34</version>
</dependency>
```

### jdbc流程

从例子中可以看出，一个基本的jdbc的程序需要四步：

1. 注册jdbc的driver
2. 获取一个连接
3. 进行查询
4. 获取结果



## 0x03 Driver注册流程分析

**Class.forName是个什么东西？**

我一直很好奇jdbc是怎么注册的，但是第一次看源码的时候对什么反射之类的概念的基本都弄不明白，因此卡在这第一行代码就卡了好久，结果好几次都没有进行下去。

```
Class.forName("com.mysql.jdbc.Driver");
```

具体的这一行的代码语法方面我还是不解释了，搞java的，Class类的功能还是需要知道的，不懂的还是需要再看看书了。

代码先继续分析，然后扭头来讲这句的功能。

**DriverManager类**

> The basic service for managing a set of JDBC drivers.

这样说吧，DriverManager是管理一个jdbc driver的基础服务。我们顺着代码看一下getConnection()方法，在源码中有着四个：

1. getConnection(String url,String user, String password)
2. getConnection(String url,java.util.Properties info)
3. getConnection(String url)
4. getConnection(String url, java.util.Properties info, ClassLoader callerCL)

其实前三个方法最后都调用了最后一个，举个栗子看一下，在我们刚才写的程序中是通过调用第一个方法来传递参数的，这个方法做了一些拼接后也是调用了最后一个方法。

```
    public static Connection getConnection(String url, String user, String password) throws SQLException {
        java.util.Properties info = new java.util.Properties();
        ClassLoader callerCL = DriverManager.getCallerClassLoader();
        if (user != null) {
            info.put("user", user);
        }
        if (password != null) {
            info.put("password", password);
        }
        //调了这个getConnection
        return (getConnection(url, info, callerCL));
    }
```

**方法：`getConnection(String url, java.util.Properties info, ClassLoader callerCL)`**

在这个方法中可以看出，当执行getConnection后，会先锁住DriverManager，然后开始registeredDrivers中所有的已注册驱动，也就是说DriverManager中注册不止一个Driver驱动，如果一些系统中有多个驱动的时候，然后它循环扫描所有的驱动程序，然后通过connect方法获取是否来调用。具体如何来进行connect后面再说，这次来详细分析Driver的注册过程。

```
    //registeredDrivers存放的已经注册的驱动。
    private final static CopyOnWriteArrayList<DriverInfo> registeredDrivers = new CopyOnWriteArrayList<DriverInfo>();

    ......

    private static Connection getConnection(
        String url, java.util.Properties info, ClassLoader callerCL) throws SQLException {
        ClassLoader callerCL = caller != null ? caller.getClassLoader() : null;
        //先锁住该类
        synchronized (DriverManager.class) {
            if (callerCL == null) {
                callerCL = Thread.currentThread().getContextClassLoader();
            }
        }
        //判断是否为空，把url设置成null试试，就抛出这个异常
        if(url == null) {
            throw new SQLException("The url cannot be null", "08001");
        }
        ......
        for(DriverInfo aDriver : registeredDrivers) {
            if(isDriverAllowed(aDriver.driver, callerCL)) {
                try {
                    //关键在这一句中，在这里尝试链接具体的url
                    Connection con = aDriver.driver.connect(url, info);
                    if (con != null) {
                        // Success!
                        println("getConnection returning " + aDriver.driver.getClass().getName());
                        return (con);
                    }
                } catch (SQLException ex) {
                    ......
                }
            }
            ......
        }
    ......
    }
```

我们先看一下Driver的注册过程。registeredDrivers是一个CopyOnWriteArrayList，它是线程安全的ArrayList，在java的concurrent包中，这是在java7后改进的，之前的版本用的不是它，具体用的是什么数据结构忘掉了。这里的驱动注册，其实就是把具体的Driver给添加到了registeredDrivers里面。下面的registerDriver方法就是把驱动的注册的过程。



```
public static synchronized void registerDriver(java.sql.Driver driver)
        throws SQLException {

        /* Register the driver if it has not already been added to our list */
        if(driver != null) {
            registeredDrivers.addIfAbsent(new DriverInfo(driver));
        } else {
            // This is for compatibility with the original DriverManager
            throw new NullPointerException();
        }
        println("registerDriver: " + driver);
    }
```
**哪些驱动会被注册？**

那么在哪个地方调用了registerDriver方法呢？这个需要继续往下看？在具体看某个特定的驱动是如何加载之前，先看一下到底有哪些驱动会被加载，它是在什么时候被加载。执行一下下面的代码。

```
//Class.forName("com.mysql.jdbc.Driver");
Enumeration<Driver> enumeration = DriverManager.getDrivers();
while (enumeration.hasMoreElements()) {
    System.out.println(enumeration.nextElement());
}

```

可以看出，在默认的时候，是有一个驱动已经被加载了的，暂时不清楚它的作用。

```
sun.jdbc.odbc.JdbcOdbcDriver@73ae9565
```

然后在把`Class.forName`的注释去掉，再执行，就可以看到多了mysql的两个驱动，我们用到的是第三个。可以看出来，`Class.forName`执行后的确是把驱动加载进来了。

```
sun.jdbc.odbc.JdbcOdbcDriver@1dd1702e
com.mysql.fabric.jdbc.FabricMySQLDriver@5aa6343d
com.mysql.jdbc.Driver@7e14feea
```

**mysql jdbc的Driver**

前面毕竟是通过结果来推测而来，下面我们进入mysql的Driver中看一下Driver究竟是如何注册的。

可以看出，在Driver这里的静态代码块里，调用了DriverManager的registerDriver方法来进行注册。也就是说，当我们在程序中调用`Class.forName("com.mysql.jdbc.Driver")`的后，com.mysql.jdbc.Driver类就会被加载，同时也在静态代码块中完成了向DriverManager的注册。这样就能回答前面提出的两个问题了吧。

```
public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    // Register ourselves with the DriverManager
    static {
        try {
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }

    //Construct a new driver and register it with DriverManager
    public Driver() throws SQLException {
        // Required for Class.forName().newInstance()
    }
}
```

**h2 jdbc的Driver**

看完mysql的顺便看一下h2的是不是也是这个样子的，多看一下又不会怀孕。可以看出来，它和mysql基本一样，都在静态代码块中通过调用DriverManager的registerDriver方法来进行注册。

```
    private static final Driver INSTANCE = new Driver();
    private static volatile boolean registered;
    static {
        load();
    }   
    public static synchronized Driver load() {
        try {
            if (!registered) {
                registered = true;
                DriverManager.registerDriver(INSTANCE);
            }
        } catch (SQLException e) {
            DbException.traceThrowable(e);
        }
        return INSTANCE;
    }
```

**NonRegisteringDriver**

NonRegisteringDriver是一个挺重要的类，后面会详细的分析它，这次就先看一下官方的注释。描述的很好很清晰。

> The Java SQL framework allows for multiple database drivers. Each driver should supply a class that implements the Driver interface

>The DriverManager will try to load as many drivers as it can find and then for any given connection request, it will ask each driver in turn to try to connect to the target URL.

>It is strongly recommended that each Driver class should be small and standalone so that the Driver class can be loaded and queried without bringing in vastquantities of supporting code.

>When a Driver class is loaded, it should create an instance of itself and register it with the DriverManager. This means that a user can load and register a

>driver by doing Class.forName("foo.bah.Driver")

## 0x04 总结

现在从头再梳理一下jdbc的注册过程。

**第一步：**

先看一下自己写的代码：

```
Class.forName("com.mysql.jdbc.Driver");
```

**第二步：**

我们写了这行代码，但是从表面上来看，其实什么都没有做。然后看一下`com.mysql.jdbc.Driver`这个类。

在`com.mysql.jdbc.Driver`中的下面这段代码中进行了驱动的注册。

```
java.sql.DriverManager.registerDriver(new Driver());
```
**第三步：**

然后在`java.sql.DriverManager.registerDriver`方法中，mysql的jdbc driver就被注册到`registeredDrivers`这个特殊的list中。


**第四步：**

然后在当需要获取connection的时候，在`DriverManager`中的一个`getConnection`中，就会从`registeredDrivers`中来获取已经注册的driver。

******
2016-06-07 17:00:00 hzct
