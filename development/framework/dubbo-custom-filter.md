---
title: dubbo-custom-filter
date: 2021-01-31 11:00:00
tags: '调度'
categories:
  - ['开发', '框架']
permalink: dubbo-custom-filter
---

https://segmentfault.com/a/1190000023066492

# [dubbo- 自定义日志拦截器](/a/1190000023066492)

[! [](https://image-static.segmentfault.com/317/931/3179314346-5f61e47221e07)**青乡之 b**](/u/qingxiangzhib) 发布于 2020-07-01

# 背景

目前的项目, 远程服务调用全部都是基于 dubbo, 有的是部门内部互相调用, 有的是调用其他部门的服务. 由于业务里面涉及到远程调用服务的地方比较多, 目前调用每个服务的时候都要手动写打印入参, 响应和异常, 比较麻烦.

现在对这块进行优化, 目的是实现自动打印入参, 响应和异常, 从而避免每个服务都要手动写重复的代码.

# 实现

## 1. 实现过滤器

新建包

XXX.solid.filter // 以 filter 结尾

新建类 - 自定义服务消费者日志拦截器

```
package XXX.solid.filter;

import com.alibaba.dubbo.common.Constants;
import com.alibaba.dubbo.common.extension.Activate;
import com.alibaba.dubbo.rpc.*;
import com.alibaba.dubbo.rpc.service.GenericService;
import com.alibaba.fastjson.JSON;
import lombok.extern.slf4j.Slf4j;

/**
 * 服务消费者日志拦截器, 作用是打印 dubbo 调用的入参和响应
 * @author gongzhihao
 */
@Slf4j
@Activate(group = {Constants.CONSUMER})
public class LogDubboConsumerFilter implements Filter {

    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        // 打印入参
        log.info("dubbo 入参, InterfaceName= {}, MethodName= {}, Parameter= {}", invocation.getInvoker().getInterface().getName(), invocation.getMethodName(), invocation.getArguments());
        // 开始时间
        long startTime = System.currentTimeMillis();
        // 执行接口调用逻辑
        Result result = invoker.invoke(invocation);
        // 调用耗时
        long elapsed = System.currentTimeMillis() - startTime;
        // 如果发生异常, 则打印异常日志
        if (result.hasException() && invoker.getInterface() != GenericService.class) {
//            log.error("dubbo 执行异常: ", result.getException());
            log.error("dubbo 执行异常!!!");
        } else {
            log.info("dubbo 响应, InterfaceName= {}, MethodName= {}, Resposne= {}, SpendTime= {} ms", invocation.getInvoker().getInterface().getName(), invocation.getMethodName(), JSON.toJSONString(new Object[]{result.getValue()}), elapsed);
        }
        // 返回结果响应数据
        return result;

    }
}
```

新建类 - 自定义服务提供者日志拦截器

```
package XXX.solid.filter;

import com.alibaba.dubbo.common.Constants;
import com.alibaba.dubbo.common.extension.Activate;
import com.alibaba.dubbo.rpc.*;
import com.alibaba.dubbo.rpc.service.GenericService;
import com.alibaba.fastjson.JSON;
import lombok.extern.slf4j.Slf4j;

/**
 * 服务提供者日志拦截器, 作用是打印 dubbo 调用的入参和响应
 * @author gongzhihao
 */
@Slf4j
@Activate(group = { Constants.PROVIDER })
public class LogDubboProviderFilter implements Filter {

    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        // 打印入参
        log.info("dubbo 入参, InterfaceName= {}, MethodName= {}, Parameter= {}", invocation.getInvoker().getInterface().getName(), invocation.getMethodName(), invocation.getArguments());
        // 开始时间
        long startTime = System.currentTimeMillis();
        // 执行接口调用逻辑
        Result result = invoker.invoke(invocation);
        // 调用耗时
        long elapsed = System.currentTimeMillis() - startTime;
        // 如果发生异常, 则打印异常日志
        if (result.hasException() && invoker.getInterface() != GenericService.class) {
//            log.error("dubbo 响应, dubbo 执行异常: ", result.getException());
            log.error("dubbo 执行异常!!!");
        } else {
            log.info("dubbo 响应, InterfaceName= {}, MethodName= {}, Response= {}, SpendTime= {} ms", invocation.getInvoker().getInterface().getName(), invocation.getMethodName(), JSON.toJSONString(new Object[]{result.getValue()}), elapsed);
        }
        // 返回结果响应数据
        return result;

    }
}
```

## 2. 配置

一, 配置

新建目录和文件: META-INF/dubbo/org.apache.dubbo.rpc.Filter

添加配置内容:

```
logDubboProviderFilter= XXX.solid.filter.LogDubboProviderFilter
logDubboConsumerFilter= XXX.solid.filter.LogDubboConsumerFilter
```

二, 启用

启用过滤器有两种方法

1. 基于注解

```
@Activate(group = { Constants.PROVIDER })
```

2. 基于配置文件

```
1.springboot 项目 -application.properties 配置文件
dubbo.provider.filter= logDubboProviderFilter
dubbo.consumer.filter= logDubboConsumerFilter


2.spring 项目 -xxx.xml 配置文件

<dubbo: provider filter="logDubboProviderFilter"/>

<dubbo: consumer retries="0" timeout="60000"  loadbalance="roundrobin" actives="0"
                validation="false" filter="logDubboConsumerFilter"/>
```

# 关闭 dubbo 自带的日志拦截器

自带的 accesslog 配置, 只打印入参, 且只在服务提供者打印入参.

如果有 accesslog 配置, 则去掉, 避免打印重复日志.

# 异常

1.dubbo 有自带的异常拦截器, 使用默认的自带的异常拦截器即可, 不做改动.

2. 如果需要在异常日志里打印业务信息 (比如, 添加异常描述信息或者通过订单 id 定位是哪一笔订单异常), 也可以自己捕获异常打印日志, 然后抛出异常.

# 参考

[http://dubbo.apache.org/zh-cn...](http://dubbo.apache.org/zh-cn/docs/dev/impls/filter.html)

[dubbo](/t/dubbo)

阅读 548 更新于 2020-07-12

赞 1 收藏

[分享](#)

本作品系原创, [采用 <署名 - 非商业性使用 - 禁止演绎 4.0 国际> 许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/)
