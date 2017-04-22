---
layout: post
title:  "Java：HttpEntity转为List<JavaBean>"
categories: 代码之熵
tags: Java
---

* content
{:toc}

## **前言**

继续之前的工作，获取Impala的接口信息后，将其转换为`List<JavaBean>`的形式，方便后续程序处理。




## **Code**

删除大部分，留点主要的处理过程。

### **JavaBean**

首先要对应一些java类，可以直接借鉴cm java api的model。

有三个类，其中TaskBase最重要。


```
import java.util.concurrent.TimeUnit;

/**
 * Task信息实体类（和接口中获取的信息对应）
 * Created by Dante on 2016/3/25.
 */
public class TaskBase {
    private String queryId;
    private String statement;
    private String queryType;
    private String queryState;
    private String startTime;
    private String endTime;
    private String rowsProduced;
    private String user;
    private String detailsAvailable;
    private String database;
    private Long durationMillis;
    private Long durationMinutes;   //此处有诡异
    private Coordinator coordinator;
    private Attributes attributes;
}
```

```
import com.google.gson.annotations.SerializedName;

/**
 * 从接口中获取到的Query信息Attributes实体类。
 * query的部分信息
 * Created by Dante on 2016/3/25.
 */
public class Attributes {

    @SerializedName("thread_storage_wait_time")
    private String threadStorageWaitTime;
    @SerializedName("session_id")
    private String sessionId;
    @SerializedName("planning_wait_time")
    private String planningWaitTime;
    @SerializedName("thread_total_time")
    private String threadTotalTime;
    @SerializedName("stats_missing")
    private String statsMissing;
    @SerializedName("thread_network_send_wait_time_percentage")
    private String threadNetworkSendWaitTimePercentage;
    @SerializedName("thread_cpu_time_percentage")
    private String threadCpuTimePercentage;
    @SerializedName("thread_network_receive_wait_time_percentage")
    private String threadNetworkReceiveWaitTimePercentage;
    @SerializedName("file_formats")
    private String fileFormats;
    @SerializedName("planning_wait_time_percentage")
    private String planningWaitTimePercentage;
    @SerializedName("client_fetch_wait_time")
    private String clientFetchWaitTime;
    @SerializedName("client_fetch_wait_time_percentage")
    private String clientFetchWaitTimePercentage;
    @SerializedName("pool")
    private String pool;
    @SerializedName("session_type")
    private String sessionType;
    @SerializedName("connected_user")
    private String connectedUser;
    @SerializedName("thread_network_receive_wait_time")
    private String threadNetworkReceiveWaitTime;
    @SerializedName("cm_cpu_milliseconds")
    private String cmCpuMilliseconds;
    @SerializedName("impala_version")
    private String impalaVersion;
    @SerializedName("thread_network_send_wait_time")
    private String threadNetworkSendWaitTime;
    @SerializedName("network_address")
    private String networkAddress;
    @SerializedName("query_status")
    private String queryStatus;
    @SerializedName("estimated_per_node_peak_memory")
    private String estimatedPerNodePeakMemory;
    @SerializedName("thread_storage_wait_time_percentage")
    private String threadStorageWaitTimePercentage;
    @SerializedName("thread_cpu_time")
    private String threadCpuTime;
}

```
```
/**
 * 从接口中获取到的Query信息Coordinator的实体类
 * Coordinator的hostID
 * Created by Dante on 2016/3/25.
 */
public class Coordinator {
    private String hostId;

    public String getHostId() {
        return hostId;
    }

    public void setHostId(String hostId) {
        this.hostId = hostId;
    }
}

```

### **调用接口类**

#### 接口
```
import com.lagou.impala.dante.entity.TaskBase;

import java.util.List;

/**
 * Task的操作相关接口
 * Created by Dante on 2016/3/25.
 */
public interface ITaskTools {
    /**
     * 和cm的管理界面相似，返回最近的几个查询。没有参数
     */
    public List<TaskBase> getRecent();

}
```


#### 实现

```
/**
 * Task执行实现类
 * cm接口太坑，很多filter没有用，不能加限制条件
 *  @// TODO: 2016/3/25 : 更改为http连接池的形式，提高性能。
 * Created by Dante on 2016/3/25.
 */
public class DefaultTaskImpl implements ITaskTools{

    CredentialsProvider credsProvider = new BasicCredentialsProvider();

    String username = "";
    String password = "";
    String queryBase = "";
    //区分从接口获得数据的queries和warning
    String queryMemberName = "";


 @Override
    public List<TaskBase> getRecent() {
        //设置认证
        credsProvider.setCredentials(new AuthScope("", AuthScope.ANY_PORT), new UsernamePasswordCredentials(username, password));
        CloseableHttpClient httpclient = HttpClients.custom().setDefaultCredentialsProvider(credsProvider).build();
        //get
        HttpGet httpget = new HttpGet(queryBase);
        List<TaskBase> taskBasesList = new LinkedList<>();
        CloseableHttpResponse response = null;
        try {
            response = httpclient.execute(httpget);
            HttpEntity entity = response.getEntity();
            //将HttpEntity装换为List<JavaBean>
            taskBasesList =  JsonUtil.httpEntityToJsonArray(entity, queryMemberName);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return taskBasesList;
    }
}
```


### **HttpEntity转List<JavaBean>**

```
import com.google.gson.*;
import com.lagou.impala.dante.entity.TaskBase;
import org.apache.http.HttpEntity;
import org.apache.http.util.EntityUtils;

import java.io.IOException;
import java.util.LinkedList;
import java.util.List;

/**
 * Created by Dante on 2016/3/25.
 */
public class JsonUtil {


    private static Gson gson = new Gson();
    private static JsonParser jp = new JsonParser();


    /**
     * 从HttpEntity获取数据，并转换为相应的List<JavaBean>
     */
    public static List<TaskBase> httpEntityToJsonArray(HttpEntity entity, String queryMemberName) {
        List<TaskBase> taskBaseList = new LinkedList<>();
        try {
            //将HttpEntity装换为List<JavaBean>
            if (entity != null) {
                //先转为String
                String retSrc = EntityUtils.toString(entity);
                //转为JsonElement
                JsonElement je = jp.parse(retSrc);
                //通过JsonElement转为JsonObject
                JsonObject ja = je.getAsJsonObject();
                //根据queryMemberName 得到JsonArray
                JsonArray jaa = ja.getAsJsonArray(queryMemberName);

                for (JsonElement joo : jaa) {
                    taskBaseList.add(gson.fromJson(joo, TaskBase.class));
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return taskBaseList;
    }

}

```

### **小测试**

#### code
```
    @org.junit.Test
    public void getRecent(){
        ITaskTools itt = new DefaultTaskImpl();
        List<TaskBase> taskBaseList = itt.getRecent();
        for(TaskBase tt :taskBaseList) {
            System.out.println(tt.getQueryState());
            System.out.println(tt.getDurationMillis());
            System.out.println(tt.getDurationMinutes());
        }
    }
```


#### 结果

```
CREATED
6247021
104
CREATED
994984
16
FINISHED
621
0

```


## 总结

CM的rest api比较坑，虽说大部分功能都没什么问题，但是一些细节明显不够，比如说文档里面的额filter，很多条件都没有用，比如说只获取正在运行的任务，这点使用rest是获取不了的，怎么调试都没通，所以只能先获取最近的所有任务，然后再筛选，总感觉这样很sb......


***
2016-04-18 19:36:00 hzct
