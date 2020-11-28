---
title: HttpClient 释放资源
date: 2020-11-21 15:00:00
tags: 'HttpClient'
categories:
  - ['开发', ' 框架']
permalink: http-client-close
---

## 简介

HttpClient 通过连接池提高性能, 关于连接池的关闭资源的方法需要了解才能更好的使用

简单了解了下默认配置 createDefault() 与 custom() 连接池的不同, 默认配置会在每次调用的时候新建一个最大连接 20, 单个路由最大为 2 的连接池, 使用 custom() 创建的连接池都是公用的, 使用时要避免每次重新建立连接池, 关闭连接池的问题

<!-- more -->

## 资源释放

```java
CloseableHttpClient httpclient = HttpClients.createDefault();
HttpGet httpget = new HttpGet("http://127.0.0.1:8080/");
CloseableHttpResponse response = httpclient.execute(httpget);
try {
    HttpEntity entity = response.getEntity();
    if (entity != null) {
        InputStream instream = entity.getContent();
        try {
            // do something useful
        } finally {
            instream.close();
        }
    }
} finally {
    response.close();
}
// httpclient.close();
```

`instream.close()` 是读取 http 正文的数据流, 类似的还有响应写入流, 都需要主动关闭, 如果使用, 使用 `EntityUtils.toString(response.getEntity(), "UTF-8");` 方法处理数据的时候, 内部逻辑会自动进行关闭

没有主动关闭将会导致 http 请求无法被复用, 由于使用的是默认的连接池, 会造成无限制等待

```sh
2020-11-25 10:18:38.241 DEBUG org.apache.http.impl.conn.PoolingHttpClientConnectionManager - Connection request: [route: {}->http://127.0.0.1:8080][total kept alive: 0; route allocated: 0 of 2; total allocated: 0 of 20]
2020-11-25 10:18:38.295 DEBUG org.apache.http.impl.conn.PoolingHttpClientConnectionManager - Connection leased: [id: 0][route: {}->http://127.0.0.1:8080][total kept alive: 0; route allocated: 1 of 2; total allocated: 1 of 20]
2020-11-25 10:18:38.298 DEBUG org.apache.http.impl.execchain.MainClientExec - Opening connection {}->http://127.0.0.1:8080
2020-11-25 10:18:38.301 DEBUG org.apache.http.impl.conn.DefaultHttpClientConnectionOperator - Connecting to /127.0.0.1:8080
2020-11-25 10:18:38.304 DEBUG org.apache.http.impl.conn.DefaultHttpClientConnectionOperator - Connection established 127.0.0.1:64931<->127.0.0.1:8080
2020-11-25 10:18:38.304 DEBUG org.apache.http.impl.execchain.MainClientExec - Executing request POST /MSCenterWeb/api/test HTTP/1.1
2020-11-25 10:18:38.304 DEBUG org.apache.http.impl.execchain.MainClientExec - Target auth state: UNCHALLENGED
2020-11-25 10:18:38.305 DEBUG org.apache.http.impl.execchain.MainClientExec - Proxy auth state: UNCHALLENGED
2020-11-25 10:18:39.597 DEBUG org.apache.http.impl.execchain.MainClientExec - Connection can be kept alive for 20000 MILLISECONDS
...
2020-11-25 10:18:50.132 DEBUG org.apache.http.impl.conn.PoolingHttpClientConnectionManager - Connection request: [route: {}->http://127.0.0.1:8080][total kept alive: 0; route allocated: 1 of 2; total allocated: 1 of 20]
2020-11-25 10:18:50.133 DEBUG org.apache.http.impl.conn.PoolingHttpClientConnectionManager - Connection leased: [id: 1][route: {}->http://127.0.0.1:8080][total kept alive: 0; route allocated: 2 of 2; total allocated: 2 of 20]
2020-11-25 10:18:50.133 DEBUG org.apache.http.impl.execchain.MainClientExec - Opening connection {}->http://127.0.0.1:8080
2020-11-25 10:18:50.133 DEBUG org.apache.http.impl.conn.DefaultHttpClientConnectionOperator - Connecting to /127.0.0.1:8080
2020-11-25 10:18:50.134 DEBUG org.apache.http.impl.conn.DefaultHttpClientConnectionOperator - Connection established 127.0.0.1:64961<->127.0.0.1:8080
2020-11-25 10:18:50.134 DEBUG org.apache.http.impl.execchain.MainClientExec - Executing request POST /MSCenterWeb/api/test2 HTTP/1.1
2020-11-25 10:18:50.134 DEBUG org.apache.http.impl.execchain.MainClientExec - Target auth state: UNCHALLENGED
2020-11-25 10:18:50.134 DEBUG org.apache.http.impl.execchain.MainClientExec - Proxy auth state: UNCHALLENGED
2020-11-25 10:18:50.156 DEBUG org.apache.http.impl.execchain.MainClientExec - Connection can be kept alive for 20000 MILLISECONDS
...
2020-11-25 10:19:10.704 DEBUG org.apache.http.impl.conn.PoolingHttpClientConnectionManager - Connection request: [route: {}->http://127.0.0.1:8080][total kept alive: 0; route allocated: 2 of 2; total allocated: 2 of 20]
```

可以看到在没有主动关闭输入流的情况下会导致一直在等待空闲链接

如果没有主动关闭输入流直接关闭请求会导致请求无法被复用

```sh
2020-11-25 11:25:47.728 DEBUG org.apache.http.impl.conn.PoolingHttpClientConnectionManager - Connection request: [route: {}->http://127.0.0.1:8080][total kept alive: 0; route allocated: 0 of 2; total allocated: 0 of 20]
2020-11-25 11:25:47.730 DEBUG org.apache.http.impl.conn.PoolingHttpClientConnectionManager - Connection leased: [id: 1][route: {}->http://127.0.0.1:8080][total kept alive: 0; route allocated: 1 of 2; total allocated: 1 of 20]
2020-11-25 11:25:47.730 DEBUG org.apache.http.impl.execchain.MainClientExec - Opening connection {}->http://127.0.0.1:8080
2020-11-25 11:25:47.731 DEBUG org.apache.http.impl.conn.DefaultHttpClientConnectionOperator - Connecting to /127.0.0.1:8080
2020-11-25 11:25:47.731 DEBUG org.apache.http.impl.conn.DefaultHttpClientConnectionOperator - Connection established 127.0.0.1:58927<->127.0.0.1:8080
2020-11-25 11:25:47.731 DEBUG org.apache.http.impl.execchain.MainClientExec - Executing request POST /MSCenterWeb/api/test HTTP/1.1
2020-11-25 11:25:47.731 DEBUG org.apache.http.impl.execchain.MainClientExec - Target auth state: UNCHALLENGED
2020-11-25 11:25:47.732 DEBUG org.apache.http.impl.execchain.MainClientExec - Proxy auth state: UNCHALLENGED
2020-11-25 11:25:49.074 DEBUG org.apache.http.impl.execchain.MainClientExec - Connection can be kept alive for 20000 MILLISECONDS
2020-11-25 11:25:49.080 DEBUG org.apache.http.impl.conn.DefaultManagedHttpClientConnection - http-outgoing-1: Close connection
2020-11-25 11:25:49.080 DEBUG org.apache.http.impl.execchain.MainClientExec - Connection discarded
2020-11-25 11:25:49.081 DEBUG org.apache.http.impl.conn.PoolingHttpClientConnectionManager - Connection released: [id: 1][route: {}->http://127.0.0.1:8080][total kept alive: 0; route allocated: 0 of 2; total allocated: 0 of 20]
```

正常调用过程中可以看到以下日志, 可以看到最后存在一个连接

```sh
2020-11-25 15:38:05.583 DEBUG org.apache.http.impl.conn.PoolingHttpClientConnectionManager - Connection request: [route: {}->http://127.0.0.1:8080][total kept alive: 0; route allocated: 0 of 2; total allocated: 0 of 20]
2020-11-25 15:38:05.649 DEBUG org.apache.http.impl.conn.PoolingHttpClientConnectionManager - Connection leased: [id: 0][route: {}->http://127.0.0.1:8080][total kept alive: 0; route allocated: 1 of 2; total allocated: 1 of 20]
2020-11-25 15:38:05.656 DEBUG org.apache.http.impl.execchain.MainClientExec - Opening connection {}->http://127.0.0.1:8080
2020-11-25 15:38:05.672 DEBUG org.apache.http.impl.conn.DefaultHttpClientConnectionOperator - Connecting to /127.0.0.1:8080
2020-11-25 15:38:05.680 DEBUG org.apache.http.impl.conn.DefaultHttpClientConnectionOperator - Connection established 127.0.0.1:50473<->127.0.0.1:8080
2020-11-25 15:38:05.682 DEBUG org.apache.http.impl.execchain.MainClientExec - Executing request POST /MSCenterWeb/api/test2 HTTP/1.1
2020-11-25 15:38:05.682 DEBUG org.apache.http.impl.execchain.MainClientExec - Target auth state: UNCHALLENGED
2020-11-25 15:38:05.687 DEBUG org.apache.http.impl.execchain.MainClientExec - Proxy auth state: UNCHALLENGED
2020-11-25 15:38:07.142 DEBUG org.apache.http.impl.execchain.MainClientExec - Connection can be kept alive for 20000 MILLISECONDS
2020-11-25 15:38:07.163 DEBUG org.apache.http.impl.conn.PoolingHttpClientConnectionManager - Connection [id: 0][route: {}->http://127.0.0.1:8080] can be kept alive for 20.0 seconds
2020-11-25 15:38:07.164 DEBUG org.apache.http.impl.conn.PoolingHttpClientConnectionManager - Connection released: [id: 0][route: {}->http://127.0.0.1:8080][total kept alive: 1; route allocated: 1 of 2; total allocated: 1 of 20]
```

但是 4.5 的版本没有使用 `response.close()` 也可以正常使用, 这个与 SOF 上的描述不一样

>> The underlying HTTP connection is still held by the response object to allow the response content to be streamed directly from the network socket. In order to ensure correct deallocation of system resources, the user MUST call CloseableHttpResponse#close() from a finally clause. Please note that if response content is not fully consumed the underlying connection cannot be safely re-used and will be shut down and discarded by the connection manager

当执行 `httpclient.close()` 时将会关闭连接池和其中所有的连接, 在关闭应用时调用以释放资源
