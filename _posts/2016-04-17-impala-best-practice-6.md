---
layout: post
title:  "Impala实践之六：使用Rest Api"
categories: 数据平台
tags: impala
---

* content
{:toc}

## 前言

上次的impala状况出现后，决定自己做一套impala的管理系统，那么首先面临的一个问题就是获取impala的各种状态，比如任务执行状态。经过一天多的尝试，总结一下。

- hue：可以使用hue的脚本，hue使用python编写，其中有一个beeswax模块，负责任务的执行等。缺点是没发现java的api。
- cloudera manager java api：java可以调用cm原生的api，需要导入jar包。跑是跑通了，但是资料太说，目前只能通过这个接口获取集群的基本情况，不想再折腾impala那快了。
- cloudera manager 的api：cm提供了rest的api供别人调用，经过接近一天的折腾，跑通了这一块。




## Cloudera Manager API

### impala相关

和impala相关的api有下面这几个，能用上的主要是最下面带queries的几个接口：

- /clusters/{clusterName}/services/{serviceName}/commands/hueSyncDb
- /clusters/{clusterName}/services/{serviceName}/commands/impalaCreateCatalogDatabase
- /clusters/{clusterName}/services/{serviceName}/commands/impalaCreateCatalogDatabaseTables
- /clusters/{clusterName}/services/{serviceName}/commands/impalaCreateUserDir
- /clusters/{clusterName}/services/{serviceName}/commands/impalaDisableLlamaHa
- /clusters/{clusterName}/services/{serviceName}/commands/impalaDisableLlamaRm
- /clusters/{clusterName}/services/{serviceName}/commands/impalaEnableLlamaHa
- /clusters/{clusterName}/services/{serviceName}/commands/impalaEnableLlamaRm
- /clusters/{clusterName}/services/{serviceName}/impalaQueries
- /clusters/{clusterName}/services/{serviceName}/impalaQueries/{queryId}
- /clusters/{clusterName}/services/{serviceName}/impalaQueries/{queryId}/cancel
- /clusters/{clusterName}/services/{serviceName}/impalaQueries/attributes

### 调用示例



**接口说明：**

*GET*

Returns details about the query. Not all queries have details, check the detailsAvailable field from the getQueries response.

```
/api/v9/clusters/{clusterName}/services/{serviceName}/impalaQueries/{queryId}
```

**注意：**
1. username:password：是指登录cm的账号和密码
2. Cluster1：是你的集群的名字
3. impala：是你的impala服务的名字
4. 364e76f05cdffsdf802b0:3a499cc3fb86bf8afsd： 是query的id

**curl例子：**
```
curl -u username:password 'clouderamanager.zhdd.com/api/v9/clusters/Cluster1/services/impala/impalaQueries/364e76f05cdffsdf802b0:3a499cc3fb86bf8afsd'
```

**结果：**

结果是一个大的json串

**java示例：**

```
import org.apache.http.auth.AuthScope;
import org.apache.http.auth.UsernamePasswordCredentials;
import org.apache.http.client.CredentialsProvider;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.impl.client.BasicCredentialsProvider;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

public class ClientAuthentication {
    public static void main(String[] args) throws Exception {
        String username = "username";
        String password = "password";
        CredentialsProvider credsProvider = new BasicCredentialsProvider();
        credsProvider.setCredentials(
                new AuthScope("clouderamanager.zhdd.com", AuthScope.ANY_PORT),
                new UsernamePasswordCredentials(username, password));
        CloseableHttpClient httpclient = HttpClients.custom()
                .setDefaultCredentialsProvider(credsProvider)
                .build();
        try {
            HttpGet httpget = new HttpGet("http://clouderamanager.zhdd.com/api/v9/clusters/Cluster1/services/impala/impalaQueries/364e76f05cdffsdf802b0:3a499cc3fb86bf8afsd");

            System.out.println("Executing request " + httpget.getRequestLine());
            CloseableHttpResponse response = httpclient.execute(httpget);
            try {
                System.out.println("----------------------------------------");
                System.out.println(response.getStatusLine());
                System.out.println(EntityUtils.toString(response.getEntity()));
            } finally {
                response.close();
            }
        } finally {
            httpclient.close();
        }
    }
}
```

## 总结

遇到这个问题，我第一反应就是用hue的，但是python和java接口感觉太麻烦，所以就找了cm的rest api。

#### 问题

cm的api应该有一些小问题，我在curl和postman上测试都比较轻松就调通了，但是使用java的httpclient调用的时候总是出错，权限认证失败，总是不能认证权限，这个问题折腾了很久，找了几个人帮忙。最后不知道怎么的我就自己把它折腾好了。

***
2016-04-17 18:39:00 hzct
