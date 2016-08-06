---
layout: post
title:  "Jdbc源码详解（二）：获取connection"
categories: 编程语言
tags:  sql jdbc java
---

* content
{:toc}

## 0x00 前言

上一节分析了jdbc的Driver注册过程，这一节分析一下jdbc是如何获取connection的。





## 0x01 connection的建立过程


**`DriverManager.getConnection` 做了什么**

```
conn = DriverManager.getConnection("jdbc:mysql://192.168.108.145/test", "root", "root");
```

可以看出，`getConnection`方法返回的是一个Connection对象，在下面的for循环中，会遍历registeredDrivers中存放的所有的Driver，直到找到一个合适的Driver，然后进行连接。通过这个方法我已经可以知道jdbc就是这样来获取连接的，但是具体是怎么来获取这个connection，还需要详细地看后面的代码。

```
private static Connection getConnection(
        String url, java.util.Properties info, ClassLoader callerCL) throws SQLException {
    ......
        for(DriverInfo aDriver : registeredDrivers) {
            // If the caller does not have permission to load the driver then
            // skip it.
            if(isDriverAllowed(aDriver.driver, callerCL)) {
                try {
                    println("    trying " + aDriver.driver.getClass().getName());
                    Connection con = aDriver.driver.connect(url, info);
                    if (con != null) {
                        // Success!
                        println("getConnection returning " + aDriver.driver.getClass().getName());
                        return (con);
                    }
                }
    ......
    }
```

**`connect`方法**

`java.sql.Driver`接口中定义了一个connect方法，然后由具体的实现类来完成connect的功能。

我们先看一下mysql的实现。

在`com.mysql.jdbc.Driver`中，并没有详细的实现，但是它继承了NonRegisteringDriver，那么connect应该就在NonRegisteringDriver中了。

```
public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    static {
        try {
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }
    public Driver() throws SQLException {
    }
}
```


现在定位到`com.mysql.jdbc.NonRegisteringDriver`的`connect`中。

可以看出，有一句`if ((props = parseURL(url, info)) == null)`来对url进行判断，如果url不符合规则的话，就会返回一个null值，这里的url就是指我们在程序中写的`jdbc:mysql://192.168.108.145/test`这样的语句。mysql的`parseURL`方法十分十分的长，暂时不再细究它了...

继续向下看，可以发现具体的获取连接的地方在` com.mysql.jdbc.ConnectionImpl.getInstance`。

```
public java.sql.Connection connect(String url, Properties info) throws SQLException {
        if (url != null) {
            if (StringUtils.startsWithIgnoreCase(url, LOADBALANCE_URL_PREFIX)) {
                return connectLoadBalanced(url, info);
            } else if (StringUtils.startsWithIgnoreCase(url, REPLICATION_URL_PREFIX)) {
                return connectReplicationConnection(url, info);
            }
        }
        Properties props = null;
        if ((props = parseURL(url, info)) == null) {
            return null;
        }

        if (!"1".equals(props.getProperty(NUM_HOSTS_PROPERTY_KEY))) {
            return connectFailover(url, info);
        }

        try {
            Connection newConn = com.mysql.jdbc.ConnectionImpl.getInstance(host(props), port(props), props, database(props), url);
            return newConn;
        }
    ......
    }
```

**漫长的寻找连接的道路**

为了找到具体获取连接的地方，我们继续详细地向下找代码。

根据方法的调用关系，我们依次找到下面的四个方法，但是在最后一个方法这里线索断了一下。

- `com.mysql.jdbc.NonRegisteringDriver.connect`

- `com.mysql.jdbc.ConnectionImpl.getInstance`

- `com.mysql.jdbc.Util.handleNewInstance.handleNewInstance`

- `java.lang.reflect.Constructor.newInstance`

``

`com.mysql.jdbc.ConnectionImpl`

```
    protected static Connection getInstance(String hostToConnectTo, int portToConnectTo, Properties info, String databaseToConnectTo, String url)
            throws SQLException {
        if (!Util.isJdbc4()) {
            return new ConnectionImpl(hostToConnectTo, portToConnectTo, info, databaseToConnectTo, url);
        }

        return (Connection) Util.handleNewInstance(JDBC_4_CONNECTION_CTOR, new Object[] { hostToConnectTo, Integer.valueOf(portToConnectTo), info,
                databaseToConnectTo, url }, null);
    }
```

`com.mysql.jdbc.Util.handleNewInstance`
```
public static final Object handleNewInstance(Constructor<?> ctor, Object[] args, ExceptionInterceptor exceptionInterceptor) throws SQLException {
        try {

            return ctor.newInstance(args);
        } catch (IllegalArgumentException e) {
        ......
    }
```

`java.lang.reflect.Constructor`
```
    public T newInstance(Object ... initargs)
        throws InstantiationException, IllegalAccessException,
               IllegalArgumentException, InvocationTargetException
    {
        if (!override) {
            if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
                Class<?> caller = Reflection.getCallerClass(2);

                checkAccess(caller, clazz, null, modifiers);
            }
        }
        if ((clazz.getModifiers() & Modifier.ENUM) != 0)
            throw new IllegalArgumentException("Cannot reflectively create enum objects");
        ConstructorAccessor ca = constructorAccessor;   // read volatile
        if (ca == null) {
            ca = acquireConstructorAccessor();
        }
        return (T) ca.newInstance(initargs);
    }
```

也就是说自己找的方向有了偏差，在`com.mysql.jdbc.ConnectionImpl.getInstance`中，有一句` return (Connection) Util.handleNewInstance(JDBC_4_CONNECTION_CTOR, ......)`，看来突破口在这里了。然后找`JDBC_4_CONNECTION_CTOR`是什么。


可以看出在这里，调用了` Class.forName("com.mysql.jdbc.JDBC4Connection").getConstructor(......)`。然后需要看一下`com.mysql.jdbc.JDBC4Connection`是做什么的。

`com.mysql.jdbc.ConnectionImpl`
```
    static {
        mapTransIsolationNameToValue = new HashMap<String, Integer>(8);
        mapTransIsolationNameToValue.put("READ-UNCOMMITED", TRANSACTION_READ_UNCOMMITTED);
        mapTransIsolationNameToValue.put("READ-UNCOMMITTED", TRANSACTION_READ_UNCOMMITTED);
        mapTransIsolationNameToValue.put("READ-COMMITTED", TRANSACTION_READ_COMMITTED);
        mapTransIsolationNameToValue.put("REPEATABLE-READ", TRANSACTION_REPEATABLE_READ);
        mapTransIsolationNameToValue.put("SERIALIZABLE", TRANSACTION_SERIALIZABLE);

        if (Util.isJdbc4()) {
            try {
                JDBC_4_CONNECTION_CTOR = Class.forName("com.mysql.jdbc.JDBC4Connection").getConstructor(
                        new Class[] { String.class, Integer.TYPE, Properties.class, String.class, String.class });
            } catch (SecurityException e) {
                throw new RuntimeException(e);
            } catch (NoSuchMethodException e) {
                throw new RuntimeException(e);
            } catch (ClassNotFoundException e) {
                throw new RuntimeException(e);
            }
        } else {
            JDBC_4_CONNECTION_CTOR = null;
        }
    }
```

继续找到了`com.mysql.jdbc.JDBC4Connection`，该类继承了`com.mysql.jdbc.ConnectionImpl`，因此在构造的时候也会先调用父类的构造器。

```
public class JDBC4Connection extends ConnectionImpl {
    private JDBC4ClientInfoProvider infoProvider;

    public JDBC4Connection(String hostToConnectTo, int portToConnectTo, Properties info, String databaseToConnectTo, String url) throws SQLException {
        super(hostToConnectTo, portToConnectTo, info, databaseToConnectTo, url);
    }
}
```


`com.mysql.jdbc.ConnectionImpl`构造器比较长，在其结尾处，发现它调用了`NonRegisteringDriver.trackConnection(this)`。
```
    public ConnectionImpl(String hostToConnectTo, int portToConnectTo, Properties info, String databaseToConnectTo, String url) throws SQLException {
        ......
        NonRegisteringDriver.trackConnection(this);
    }
```

`com.mysql.jdbc.NonRegisteringDriver`

```
    protected static void trackConnection(Connection newConn) {
        ConnectionPhantomReference phantomRef = new ConnectionPhantomReference((ConnectionImpl) newConn, refQueue);
        connectionPhantomRefs.put(phantomRef, phantomRef);
    }
```


`com.mysql.jdbc.NonRegisteringDriver.ConnectionPhantomReference`是`NonRegisteringDriver`的内部类。

```
 static class ConnectionPhantomReference extends PhantomReference<ConnectionImpl> {
        private NetworkResources io;

        ConnectionPhantomReference(ConnectionImpl connectionImpl, ReferenceQueue<ConnectionImpl> q) {
            super(connectionImpl, q);

            try {
                this.io = connectionImpl.getIO().getNetworkResources();
            } catch (SQLException e) {
                // if we somehow got here and there's really no i/o, we deal with it later
            }
        }

        void cleanup() {
            if (this.io != null) {
                try {
                    this.io.forceClose();
                } finally {
                    this.io = null;
                }
            }
        }
    }
```

继续向下追，会用到两个比较重要的类：`NetworkResources`和`MysqlIO`,他们和mysql的具体连接相关。


`com.mysql.jdbc.NetworkResources`
```
class NetworkResources {
    private final Socket mysqlConnection;
    private final InputStream mysqlInput;
    private final OutputStream mysqlOutput;
    protected NetworkResources(Socket mysqlConnection, InputStream mysqlInput, OutputStream mysqlOutput) {
        this.mysqlConnection = mysqlConnection;
        this.mysqlInput = mysqlInput;
        this.mysqlOutput = mysqlOutput;
    }
}
```

最后可以看出真正和mysql进行连接是在`com.mysql.jdbc.MysqlIO`中实现的，在这里有mysql连接的各种参数。

>This class is used by Connection for communicating with the MySQL server.

```
    //Mysql连接的一些设置参数
    private static final int CLIENT_LONG_PASSWORD = 0x00000001; /* new more secure passwords */
    private static final int CLIENT_FOUND_ROWS = 0x00000002;
    private static final int CLIENT_LONG_FLAG = 0x00000004; /* Get all column flags */
    protected static final int CLIENT_CONNECT_WITH_DB = 0x00000008;
    private static final int CLIENT_COMPRESS = 0x00000020; /* Can use compression protcol */
    private static final int CLIENT_LOCAL_FILES = 0x00000080; /* Can use LOAD DATA LOCAL */
    private static final int CLIENT_PROTOCOL_41 = 0x00000200; // for > 4.1.1
    ......

    //MysqlIO构造函数
    public MysqlIO(String host, int port, Properties props, String socketFactoryClassName, MySQLConnection conn, int socketTimeout, int useBufferRowSizeThreshold) throws IOException, SQLException {

        ......

        try {
            // 创建Socket对象,和MySql服务器建立连接  
            this.mysqlConnection = this.socketFactory.connect(this.host, this.port, props);

            if (socketTimeout != 0) {
                try {
                    this.mysqlConnection.setSoTimeout(socketTimeout);
                } catch (Exception ex) {
                    /* Ignore if the platform does not support it */
                }
            }
            // 获取Socket对象
            this.mysqlConnection = this.socketFactory.beforeHandshake();

            if (this.connection.getUseReadAheadInput()) {
                this.mysqlInput = new ReadAheadInputStream(this.mysqlConnection.getInputStream(), 16384, this.connection.getTraceProtocol(),
                        this.connection.getLog());
            } else if (this.connection.useUnbufferedInput()) {
                this.mysqlInput = this.mysqlConnection.getInputStream();
            } else {
                this.mysqlInput = new BufferedInputStream(this.mysqlConnection.getInputStream(), 16384);
            }

            //封装ScoketOutputStream输出流
            this.mysqlOutput = new BufferedOutputStream(this.mysqlConnection.getOutputStream(), 16384);

            this.isInteractiveClient = this.connection.getInteractiveClient();
            this.profileSql = this.connection.getProfileSql();
            this.autoGenerateTestcaseScript = this.connection.getAutoGenerateTestcaseScript();

            this.needToGrabQueryFromPacket = (this.profileSql || this.logSlowQueries || this.autoGenerateTestcaseScript);
            //封装SocketInputStream输入流  
            if (this.connection.getUseNanosForElapsedTime() && Util.nanoTimeAvailable()) {
                this.useNanosForElapsedTime = true;

                this.queryTimingUnits = Messages.getString("Nanoseconds");
            } else {
                this.queryTimingUnits = Messages.getString("Milliseconds");
            }

            if (this.connection.getLogSlowQueries()) {
                calculateSlowQueryThreshold();
            }
        } catch (IOException ioEx) {
            throw SQLError.createCommunicationsException(this.connection, 0, 0, ioEx, getExceptionInterceptor());
        }
    }
```

## 0x02 总结


现在从头再梳理一下jdbc的注册过程。

**第一步：**

先看一下自己写的代码：

```
conn = DriverManager.getConnection("jdbc:mysql://192.168.108.145/test", "root", "root");
```

**第二步：**

`getConnection`方法中会获取到一个具体的connection。`Connection con = aDriver.driver.connect(url, info);`那么这个connection是怎么获取的呢，不同的厂商有自己特有的实现方式，对于mysql来说，在绕了一大圈后，最终调用了MysqlIO这个类来完成了和mysql-servier的连接。

**第三步：**

MysqlIO在获取到连接后，以NetworkResources的方式返回，这个NetworkResources就是mysql连接的一些基本信息。最后的连接由DriverManager的getConnection来获取。

**第四步：**

没了。

******
2016-06-14 14:23:00 hzct
