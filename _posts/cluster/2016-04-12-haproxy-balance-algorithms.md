---
layout: post
title:  "Haproxy负载均衡算法"
categories: 漫步云端
tags: Haproxy
---

* content
{:toc}

## 前言

前段时间在impala集群中用到Haproxy，秉着折腾的原则，略微整理一下Haproxy的负载均衡算法，大致看了一下，基本上包括了主流的负载均衡算法。都是英文就不翻译了。自己大致明白的算法就凑一句自己的理解，不理解和没听过的，就算了。

目前，我一直使用的是leastconn。




## 算法

### **roundrobin**

**轮询。**

这就个就是轮询了，挨个分配机器。适合于服务器组中的所有服务器都有相同的软硬件配置并且平均服务请求相对均衡的情况。

>Each server is used in turns, according to their weights.This is the smoothest and fairest algorithm when the server'sprocessing time remains equally distributed. This algorithm is dynamic, which means that server weights may be adjusted on the fly for slow starts for instance. It is limited by design to 4095 active servers per backend. Note that in some large farms, when a server becomes up after having been down for a very short time, it may sometimes take a few hundreds requests for it to be re-integrated into the farm and start receiving traffic. This is normal, though very rare. It is indicated here in case you would have the chance to observe it, so that you don't worry.

### **static-rr**

**加权轮询。**

haproxy起的这个名字也是好玩，不看注释谁能猜出来这个算法......

根据服务器的不同处理能力，给每个服务器分配不同的权值，使其能够接受相应权值数的服务请求。

>Each server is used in turns, according to their weights. This algorithm is as similar to roundrobin except that it is static, which means that changing a server's weight on the fly will have no effect. On the other hand, it has no design limitation on the number of servers, and when a server goes up, it is always immediately reintroduced into the farm, once the full map is recomputed. It also uses slightly less CPU to run (around -1%).

### **leastconn**

**最少连接数。**

最少连接数均衡算法对内部中需负载的每一台服务器都有一个数据记录，记录当前该服务器正在处理的连接数量，当有新的服务连接请求时，将把当前请求分配给连接数最少的服务器，使均衡更加符合实际情况，负载更加均衡。此种均衡算法适合长时处理的请求服务，如FTP。

>The server with the lowest number of connections receives the connection. Round-robin is performed within groups of servers of the same load to ensure that all servers will be used. Use of this algorithm is recommended where very long sessions are expected, such as LDAP, SQL, TSE, etc... but is not very well suited for protocols using short sessions such as HTTP. This algorithm is dynamic, which means that server weights may be adjusted on the fly for slow starts for instance.

### **source**      

**IP Hash。**

根据请求的源IP地址，作为散列键（Hash Key）从静态分配的散列表找出对应的服务器。

同一IP地址，达到的是同一台server。

>The source IP address is hashed and divided by the total weight of the running servers to designate which server will receive the request. This ensures that the same client IP address will always reach the same server as long as no server goes down or up. If the hash result changes due to the number of running servers changing, many clients will be directed to a different server. This algorithm is generally used in TCP mode where no cookie may be inserted. It may also be used on the Internet to provide a best-effort stickiness to clients which refuse session cookies. This algorithm is static by default, which means that changing a server's weight on the fly will have no effect, but this can be changed using "hash-type".

### **first**

这个算法挺有意思，英语解释挺清晰的。

>The first server with available connection slots receives the connection. The servers are chosen from the lowest numeric identifier to the highest (see server parameter "id"), which defaults to the server's position in the farm. Once a server reaches its maxconn value, the next server is used. It does not make sense to use this algorithm without setting maxconn. The purpose of this algorithm is to always use the smallest number of servers so that extra servers can be powered off during non-intensive hours. This algorithm ignores the server weight, and brings more benefit to long session such as RDP or IMAP than HTTP, though it can be useful there too. In order to use this algorithm efficiently, it is recommended that a cloud controller regularly checks server usage to turn them off when unused, and regularly checks backend queue to turn new servers on when the queue inflates. Alternatively, using "http-check send-state" may inform servers on the load.




### **uri**     

也是一种hash算法，只是hash的方式换了一下而已。

>This algorithm hashes either the left part of the URI (before the question mark) or the whole URI (if the "whole" parameter is present) and divides the hash value by the total weight of the running servers. The result designates which server will receive the request. This ensures that the same URI will always be directed to the same server as long as no server goes up or down. This is used with proxy caches and anti-virus proxies in order to maximize the cache hit rate. Note that this algorithm may only be used in an HTTP backend. This algorithm is static by default, which means that changing a server's weight on the fly will have no effect, but this can be changed using "hash-type".

>This algorithm supports two optional parameters "len" and "depth", both followed by a positive integer number. These options may be helpful when it is needed to balance servers based on the beginning of the URI only. The "len" parameter indicates that the algorithm should only consider that many characters at the beginning of the URI to compute the hash. Note that having "len" set to 1 rarely makes sense since most URIs start with a leading "/".

>The "depth" parameter indicates the maximum directory depth to be used to compute the hash. One level is counted for each slash in the request. If both parameters are specified, the evaluation stops when either is reached.


### **url_param**  

>The URL parameter specified in argument will be looked up inthe query string of each HTTP GET request.

>If the modifier "check_post" is used, then an HTTP POST request entity will be searched for the parameter argument, when it is not found in a query string after a question mark ('?') in the URL. The message body will only start to be analyzed once either the advertised amount of data has been received or the request buffer is full. In the unlikely event that chunked encoding is used, only the first chunk is scanned. Parameter values separated by a chunk boundary, may be randomly balanced if at all. This keyword used to support an optional <max_wait> parameter which is now ignored.

>If the parameter is found followed by an equal sign ('=') and a value, then the value is hashed and divided by the total weight of the running servers. The result designates which server will receive the request.

>This is used to track user identifiers in requests and ensure that a same user ID will always be sent to the same server as long as no server goes up or down. If no value is found or if the parameter is not found, then a round robin algorithm is applied. Note that this algorithm may only be used in an HTTP backend. This algorithm is static by default, which means that changing a server's weight on the fly will have no effect, but this can be changed using "hash-type".

### **hdr(<name>)**

>The HTTP header <name> will be looked up in each HTTP request. Just as with the equivalent ACL 'hdr()' function, the header name in parenthesis is not case sensitive. If the header is absent or if it does not contain any value, the roundrobin algorithm is applied instead.

>An optional 'use_domain_only' parameter is available, for reducing the hash algorithm to the main domain part with some specific headers such as 'Host'. For instance, in the Host value "haproxy.1wt.eu", only "1wt" will be considered.

>This algorithm is static by default, which means that changing a server's weight on the fly will have no effect, but this can be changed using "hash-type".


### **rdp-cookie(<name>)**

>The RDP cookie <name> (or "mstshash" if omitted) will be looked up and hashed for each incoming TCP request. Just as with the equivalent ACL 'req_rdp_cookie()' function, the name is not case-sensitive. This mechanism is useful as a degraded persistence mode, as it makes it possible to always send the same user (or the same session ID) to the same server. If the cookie is not found, the normal roundrobin algorithm is used instead.


>Note that for this to work, the frontend must ensure that an RDP cookie is already present in the request buffer. For this you must use 'tcp-request content accept' rule combined with a 'req_rdp_cookie_cnt' ACL.

>This algorithm is static by default, which means that changing a server's weight on the fly will have no effect, but this can be changed using "hash-type". See also the rdp_cookie pattern fetch function.

***
2016-04-12 19:30:00 hzct
