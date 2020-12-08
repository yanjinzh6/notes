---
title: HttpClient 4.5 连接池
date: 2020-12-05 17:00:00
tags: 'Spring'
categories:
  - ['开发', '框架']
permalink: spring-cloud-refresh-scope
---

## 简介

HttpClient 通过连接池提高效率, 但是连接池配置比较关键, 并发量高的情况下, 使用默认的连接池配置可能会导致响应缓慢

<!-- more -->

## 连接池

默认 `createDefault()` 会为每个实例创建一个最大 20 个连接, 2 个路由的连接池

使用 `PoolingHttpClientConnectionManager` 类管理的连接池可以为所有的实例使用

当请求一个新的连接时, 如果连接池有可用的持久连接, 连接管理器就会使用其中的一个, 而不是再创建一个新的连接

`PoolingHttpClientConnectionManager` 维护的连接数在每个路由基础和总数上都有限制, 默认, 每个路由基础上的连接不超过 2 个, 总连接数不能超过 20, 在实际应用中, 这个限制可能会太小了, 尤其是当服务器也使用 Http 协议时

## 实现

```java
/** 全局连接池对象 */
private static final PoolingHttpClientConnectionManager connManager = new PoolingHttpClientConnectionManager();

/**
  * 静态代码块配置连接池信息
  */
static {

    // 设置最大连接数
    connManager.setMaxTotal(200);
    // 设置每个连接的路由数
    connManager.setDefaultMaxPerRoute(20);

}

/**
  * 获取 Http 客户端连接对象
  *
  * @param timeOut 超时时间
  * @return Http客户端连接对象
  */
public static CloseableHttpClient getHttpClient(Integer timeOut) {
    // 创建Http请求配置参数
    RequestConfig requestConfig = RequestConfig.custom()
        // 获取连接超时时间
        .setConnectionRequestTimeout(timeOut)
        // 请求超时时间
        .setConnectTimeout(timeOut)
        // 响应超时时间
        .setSocketTimeout(timeOut)
        .build();

    /**
      * 测出超时重试机制为了防止超时不生效而设置
      *  如果直接放回false,不重试
      *  这里会根据情况进行判断是否重试
      */
    HttpRequestRetryHandler retry = new HttpRequestRetryHandler() {
        @Override
        public boolean retryRequest(IOException exception, int executionCount, HttpContext context) {
            // 如果已经重试了 3 次, 就放弃
            if (executionCount >= 3) {
                return false;
            }
            // 如果服务器丢掉了连接, 那么就重试
            if (exception instanceof NoHttpResponseException) {
                return true;
            }
            // 不要重试 SSL 握手异常
            if (exception instanceof SSLHandshakeException) {
                return false;
            }
            // 超时
            if (exception instanceof InterruptedIOException) {
                return true;
            }
            // 目标服务器不可达
            if (exception instanceof UnknownHostException) {
                return false;
            }
            // 连接被拒绝
            if (exception instanceof ConnectTimeoutException) {
                return false;
            }
            // ssl 握手异常
            if (exception instanceof SSLException) {
                return false;
            }
            HttpClientContext clientContext = HttpClientContext.adapt(context);
            HttpRequest request = clientContext.getRequest();
            // 如果请求是幂等的, 就再次尝试
            if (!(request instanceof HttpEntityEnclosingRequest)) {
                return true;
            }
            return false;
        }
    };

    // 创建httpClient
    return HttpClients.custom()
            // 把请求相关的超时信息设置到连接客户端
            .setDefaultRequestConfig(requestConfig)
            // 把请求重试设置到连接客户端
            .setRetryHandler(retry)
            // 配置连接池管理对象
            .setConnectionManager(connManager)
            .build();
}

/**
  * GET请求
  *
  * @param url 请求地址
  * @param timeOut 超时时间
  * @return
  */
public static String httpGet(String url, Integer timeOut) {
    String msg = "-1";

    // 获取客户端连接对象
    CloseableHttpClient httpClient = getHttpClient(timeOut);
    // 创建GET请求对象
    HttpGet httpGet = new HttpGet(url);

    CloseableHttpResponse response = null;

    try {
        // 执行请求
        response = httpClient.execute(httpGet);
        // 获取响应实体
        HttpEntity entity = response.getEntity();
        // 获取响应信息
        msg = EntityUtils.toString(entity, "UTF-8");
    } catch (ClientProtocolException e) {
        System.err.println("协议错误");
        e.printStackTrace();
    } catch (ParseException e) {
        System.err.println("解析错误");
        e.printStackTrace();
    } catch (IOException e) {
        System.err.println("IO错误");
        e.printStackTrace();
    } finally {
        if (null != response) {
            try {
                EntityUtils.consume(response.getEntity());
                response.close();
            } catch (IOException e) {
                System.err.println("释放链接错误");
                e.printStackTrace();
            }
        }
    }

    return msg;
}
```
