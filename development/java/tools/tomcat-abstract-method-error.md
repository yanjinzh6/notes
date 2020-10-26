---
title: 部署到 Tomcat 8.5 容器出现 AbstractMethodError
date: 2020-09-26 19:00:00
tags: 'Tomcat'
categories:
  - ['开发', 'Java', '工具']
permalink: tomcat-abstract-method-error
---

## 简介

使用 spring-boot 可以很方便的启动一个应用, 也可以打包成 war 进行容器部署启动

```sh
# 版本
spring-boot: 2.2.5.RELEASE
tomcat: 8.5.57
```

部署后启动出现错误

```sh
严重 [RMI TCP Connection(4)-127.0.0.1] org.apache.catalina.core.StandardContext.filterStart 启动过滤器异常
 java.lang.AbstractMethodError
  at org.apache.catalina.core.ApplicationFilterConfig.initFilter(ApplicationFilterConfig.java:281)
  at org.apache.catalina.core.ApplicationFilterConfig.<init>(ApplicationFilterConfig.java:110)
  at org.apache.catalina.core.StandardContext.filterStart(StandardContext.java:4538)
  at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5181)
  at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183)
  at org.apache.catalina.core.ContainerBase.addChildInternal(ContainerBase.java:743)
  at org.apache.catalina.core.ContainerBase.addChild(ContainerBase.java:719)
  at org.apache.catalina.core.StandardHost.addChild(StandardHost.java:705)
  at org.apache.catalina.startup.HostConfig.manageApp(HostConfig.java:1719)
  at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
  at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
  at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
  at java.lang.reflect.Method.invoke(Method.java:498)
  at org.apache.tomcat.util.modeler.BaseModelMBean.invoke(BaseModelMBean.java:286)
  at com.sun.jmx.interceptor.DefaultMBeanServerInterceptor.invoke(DefaultMBeanServerInterceptor.java:819)
  at com.sun.jmx.mbeanserver.JmxMBeanServer.invoke(JmxMBeanServer.java:801)
  at org.apache.catalina.mbeans.MBeanFactory.createStandardContext(MBeanFactory.java:479)
  at org.apache.catalina.mbeans.MBeanFactory.createStandardContext(MBeanFactory.java:428)
  at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
  at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
  at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
  at java.lang.reflect.Method.invoke(Method.java:498)
  at org.apache.tomcat.util.modeler.BaseModelMBean.invoke(BaseModelMBean.java:286)
  at com.sun.jmx.interceptor.DefaultMBeanServerInterceptor.invoke(DefaultMBeanServerInterceptor.java:819)
  at com.sun.jmx.mbeanserver.JmxMBeanServer.invoke(JmxMBeanServer.java:801)
  at com.sun.jmx.remote.security.MBeanServerAccessController.invoke(MBeanServerAccessController.java:468)
  at javax.management.remote.rmi.RMIConnectionImpl.doOperation(RMIConnectionImpl.java:1468)
  at javax.management.remote.rmi.RMIConnectionImpl.access$300(RMIConnectionImpl.java:76)
  at javax.management.remote.rmi.RMIConnectionImpl$PrivilegedOperation.run(RMIConnectionImpl.java:1309)
  at java.security.AccessController.doPrivileged(Native Method)
  at javax.management.remote.rmi.RMIConnectionImpl.doPrivilegedOperation(RMIConnectionImpl.java:1408)
  at javax.management.remote.rmi.RMIConnectionImpl.invoke(RMIConnectionImpl.java:829)
  at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
  at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
  at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
  at java.lang.reflect.Method.invoke(Method.java:498)
  at sun.rmi.server.UnicastServerRef.dispatch(UnicastServerRef.java:357)
  at sun.rmi.transport.Transport$1.run(Transport.java:200)
  at sun.rmi.transport.Transport$1.run(Transport.java:197)
  at java.security.AccessController.doPrivileged(Native Method)
  at sun.rmi.transport.Transport.serviceCall(Transport.java:196)
  at sun.rmi.transport.tcp.TCPTransport.handleMessages(TCPTransport.java:573)
  at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run0(TCPTransport.java:834)
  at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.lambda$run$0(TCPTransport.java:688)
  at java.security.AccessController.doPrivileged(Native Method)
  at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run(TCPTransport.java:687)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
  at java.lang.Thread.run(Thread.java:748)
29-Sep-2020 16:39:41.201 信息 [RMI TCP Connection(4)-127.0.0.1] org.apache.catalina.core.ApplicationContext.log Closing Spring root WebApplicationContext
```

<!-- more -->

## 分析

单看错误堆栈没有出现比较明显的信息, 但是出现 AbstractMethodError 错误可以推断出是调用抽象方法时抛出异常

可以定位到出错位置

```java
// org.apache.catalina.core.ApplicationFilterConfig#initFilter
private void initFilter() throws ServletException {
    if (context instanceof StandardContext &&
            context.getSwallowOutput()) {
        try {
            SystemLogHandler.startCapture();
            filter.init(this);
        } finally {
            String capturedlog = SystemLogHandler.stopCapture();
            if (capturedlog != null && capturedlog.length() > 0) {
                getServletContext().log(capturedlog);
            }
        }
    } else {
        filter.init(this);
    }

    // Expose filter via JMX
    registerJMX();
}
```

```java
org.apache.catalina.core.ApplicationFilterConfig private transient Filter filter = null
```

可以看到是在初始化 filter 链中出现问题, 调用 Filter 接口的 init 方法出错

通过断点可以确定是哪个 filter 出错

```java
// org.apache.catalina.core.StandardContext#filterStart
/**
  * Configure and initialize the set of filters for this Context.
  * @return <code>true</code> if all filter initialization completed
  * successfully, or <code>false</code> otherwise.
  */
public boolean filterStart() {

    if (getLogger().isDebugEnabled()) {
        getLogger().debug("Starting filters");
    }
    // Instantiate and record a FilterConfig for each defined filter
    boolean ok = true;
    synchronized (filterConfigs) {
        filterConfigs.clear();
        for (Entry<String,FilterDef> entry : filterDefs.entrySet()) {
            String name = entry.getKey();
            if (getLogger().isDebugEnabled()) {
                getLogger().debug(" Starting filter '" + name + "'");
            }
            try {
                ApplicationFilterConfig filterConfig =
                        new ApplicationFilterConfig(this, entry.getValue());
                filterConfigs.put(name, filterConfig);
            } catch (Throwable t) {
                t = ExceptionUtils.unwrapInvocationTargetException(t);
                ExceptionUtils.handleThrowable(t);
                getLogger().error(sm.getString(
                        "standardContext.filterStart", name), t);
                ok = false;
            }
        }
    }

    return ok;
}
```

出现错误的类并可以运行在内嵌的 tomcat 容器中, 其他容器也可以

看了下内嵌的 tomcat 版本为: `9.0.37` 对比一下两个版本的 `Filter.class` 接口

```java
// 9.0.31
// javax.servlet.Filter
/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package javax.servlet;

import java.io.IOException;

/**
 * A filter is an object that performs filtering tasks on either the request to
 * a resource (a servlet or static content), or on the response from a resource,
 * or both. <br>
 * <br>
 * Filters perform filtering in the <code>doFilter</code> method. Every Filter
 * has access to a FilterConfig object from which it can obtain its
 * initialization parameters, a reference to the ServletContext which it can
 * use, for example, to load resources needed for filtering tasks.
 * <p>
 * Filters are configured in the deployment descriptor of a web application
 * <p>
 * Examples that have been identified for this design are<br>
 * 1) Authentication Filters <br>
 * 2) Logging and Auditing Filters <br>
 * 3) Image conversion Filters <br>
 * 4) Data compression Filters <br>
 * 5) Encryption Filters <br>
 * 6) Tokenizing Filters <br>
 * 7) Filters that trigger resource access events <br>
 * 8) XSL/T filters <br>
 * 9) Mime-type chain Filter <br>
 *
 * @since Servlet 2.3
 */
public interface Filter {

    /**
     * Called by the web container to indicate to a filter that it is being
     * placed into service. The servlet container calls the init method exactly
     * once after instantiating the filter. The init method must complete
     * successfully before the filter is asked to do any filtering work.
     * <p>
     * The web container cannot place the filter into service if the init method
     * either:
     * <ul>
     * <li>Throws a ServletException</li>
     * <li>Does not return within a time period defined by the web
     *     container</li>
     * </ul>
     * The default implementation is a NO-OP.
     *
     * @param filterConfig The configuration information associated with the
     *                     filter instance being initialised
     *
     * @throws ServletException if the initialisation fails
     */
    public default void init(FilterConfig filterConfig) throws ServletException {}

    /**
     * The <code>doFilter</code> method of the Filter is called by the container
     * each time a request/response pair is passed through the chain due to a
     * client request for a resource at the end of the chain. The FilterChain
     * passed in to this method allows the Filter to pass on the request and
     * response to the next entity in the chain.
     * <p>
     * A typical implementation of this method would follow the following
     * pattern:- <br>
     * 1. Examine the request<br>
     * 2. Optionally wrap the request object with a custom implementation to
     * filter content or headers for input filtering <br>
     * 3. Optionally wrap the response object with a custom implementation to
     * filter content or headers for output filtering <br>
     * 4. a) <strong>Either</strong> invoke the next entity in the chain using
     * the FilterChain object (<code>chain.doFilter()</code>), <br>
     * 4. b) <strong>or</strong> not pass on the request/response pair to the
     * next entity in the filter chain to block the request processing<br>
     * 5. Directly set headers on the response after invocation of the next
     * entity in the filter chain.
     *
     * @param request  The request to process
     * @param response The response associated with the request
     * @param chain    Provides access to the next filter in the chain for this
     *                 filter to pass the request and response to for further
     *                 processing
     *
     * @throws IOException if an I/O error occurs during this filter's
     *                     processing of the request
     * @throws ServletException if the processing fails for any other reason
     */
    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;

    /**
     * Called by the web container to indicate to a filter that it is being
     * taken out of service. This method is only called once all threads within
     * the filter's doFilter method have exited or after a timeout period has
     * passed. After the web container calls this method, it will not call the
     * doFilter method again on this instance of the filter. <br>
     * <br>
     *
     * This method gives the filter an opportunity to clean up any resources
     * that are being held (for example, memory, file handles, threads) and make
     * sure that any persistent state is synchronized with the filter's current
     * state in memory.
     *
     * The default implementation is a NO-OP.
     */
    public default void destroy() {}
}
```

```java
// 8.5.57
// javax.servlet.Filter
/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package javax.servlet;

import java.io.IOException;

/**
 * A filter is an object that performs filtering tasks on either the request to
 * a resource (a servlet or static content), or on the response from a resource,
 * or both. <br>
 * <br>
 * Filters perform filtering in the <code>doFilter</code> method. Every Filter
 * has access to a FilterConfig object from which it can obtain its
 * initialization parameters, a reference to the ServletContext which it can
 * use, for example, to load resources needed for filtering tasks.
 * <p>
 * Filters are configured in the deployment descriptor of a web application
 * <p>
 * Examples that have been identified for this design are<br>
 * 1) Authentication Filters <br>
 * 2) Logging and Auditing Filters <br>
 * 3) Image conversion Filters <br>
 * 4) Data compression Filters <br>
 * 5) Encryption Filters <br>
 * 6) Tokenizing Filters <br>
 * 7) Filters that trigger resource access events <br>
 * 8) XSL/T filters <br>
 * 9) Mime-type chain Filter <br>
 *
 * @since Servlet 2.3
 */
public interface Filter {

    /**
     * Called by the web container to indicate to a filter that it is being
     * placed into service. The servlet container calls the init method exactly
     * once after instantiating the filter. The init method must complete
     * successfully before the filter is asked to do any filtering work.
     * <p>
     * The web container cannot place the filter into service if the init method
     * either:
     * <ul>
     * <li>Throws a ServletException</li>
     * <li>Does not return within a time period defined by the web
     *     container</li>
     * </ul>
     *
     * @param filterConfig The configuration information associated with the
     *                     filter instance being initialised
     *
     * @throws ServletException if the initialisation fails
     */
    public void init(FilterConfig filterConfig) throws ServletException;

    /**
     * The <code>doFilter</code> method of the Filter is called by the container
     * each time a request/response pair is passed through the chain due to a
     * client request for a resource at the end of the chain. The FilterChain
     * passed in to this method allows the Filter to pass on the request and
     * response to the next entity in the chain.
     * <p>
     * A typical implementation of this method would follow the following
     * pattern:- <br>
     * 1. Examine the request<br>
     * 2. Optionally wrap the request object with a custom implementation to
     * filter content or headers for input filtering <br>
     * 3. Optionally wrap the response object with a custom implementation to
     * filter content or headers for output filtering <br>
     * 4. a) <strong>Either</strong> invoke the next entity in the chain using
     * the FilterChain object (<code>chain.doFilter()</code>), <br>
     * 4. b) <strong>or</strong> not pass on the request/response pair to the
     * next entity in the filter chain to block the request processing<br>
     * 5. Directly set headers on the response after invocation of the next
     * entity in the filter chain.
     *
     * @param request  The request to process
     * @param response The response associated with the request
     * @param chain    Provides access to the next filter in the chain for this
     *                 filter to pass the request and response to for further
     *                 processing
     *
     * @throws IOException if an I/O error occurs during this filter's
     *                     processing of the request
     * @throws ServletException if the processing fails for any other reason
     */
    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;

    /**
     * Called by the web container to indicate to a filter that it is being
     * taken out of service. This method is only called once all threads within
     * the filter's doFilter method have exited or after a timeout period has
     * passed. After the web container calls this method, it will not call the
     * doFilter method again on this instance of the filter. <br>
     * <br>
     *
     * This method gives the filter an opportunity to clean up any resources
     * that are being held (for example, memory, file handles, threads) and make
     * sure that any persistent state is synchronized with the filter's current
     * state in memory.
     */
    public void destroy();
}
```

可以看到 tomcat 9 使用了 `servlet-api:4` 版本的接口, 其中 init 和 destroy 方法都已经默认实现了

## 验证

将 `apache-tomcat-8.5.57/lib/servlet-api.jar` 替换为 `servlet-api:4.0.1` 版本, 重新启动后正常

## 解决

将出现问题的自定义 filter 实现 init 和 destroy 方法即可

## 参考

[Apache Tomcat Versions](http://tomcat.apache.org/whichversion.html)
