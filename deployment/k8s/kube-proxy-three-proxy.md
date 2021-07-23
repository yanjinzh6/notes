---
title: kube-proxy-three-proxy
date: 2021-03-14 16:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: kube-proxy-three-proxy
photo:
---

[kube-proxy-three-proxy](https://www.huaweicloud.com/articles/5c53335e9ef19f190d681ff06dcb930c.html)

[精选文章](https://www.huaweicloud.com/articles/articles-A-1.html) kube-proxy 的三种代理模式

# kube-proxy 的三种代理模式

_[作者: 三胖桑](#)_ 时间: 2019-11-06 10:23:00

 [](#)  _ [三胖桑](#) _  2019-11-06 10:23:00

[标签:](https://www.huaweicloud.com/articles/topic-A-1.html) [k8s 和 docker](http://www.huaweicloud.com/articles/topic_58969baa7a6814d6ccc6e094b39b7c54.html "k8s 和 docker")

# 前言

Service 是 k8s 中资源的一种, 也是 k8s 能够实现减少运维工作量, 甚至免运维的关键点, 我们公司的运维都要把服务搭在我们集群里, 接触过的人应该都能体会到其方便之处.Service 能将 pod 的变化屏蔽在集群内部, 同时提供负载均衡的能力, 自动将请求流量分布到后端的 pod, 这一功能的实现靠的就是 kube-proxy 的流量代理, 一共有三种模式, userspace, iptables 以及 ipvs.

# 1, userspace

网上的图是这样的:

! [kube-proxy 的三种代理模式 1](https://res-static.hc-cdn.cn/fms/img/83e89ea22b909c86210ed139b3f2f9761603447379206.png)

没大理解, 自己画的一下:

! [kube-proxy 的三种代理模式 2](https://res-static.hc-cdn.cn/fms/img/1edd58f721220c80663c71eee54009541603447379206.png)

1, 为每个 service 在 node 上打开一个随机端口 (代理端口)

2, 建立 iptables 规则, 将 clusterip 的请求重定向到代理端口

3, 到达代理端口 (用户空间) 的请求再由 kubeproxy 转发到后端 pod.

这里为什么需要建 iptables 规则, 因为 kube-proxy 监听的端口在用户空间, 所以需要一层 iptables 把访问服务的连接重定向给 kube-proxy 服务, 这里就存在内核态到用户态的切换, 代价很大, 因此就有了 iptables.

# 2, iptables

! [kube-proxy 的三种代理模式 3](https://res-static.hc-cdn.cn/fms/img/fd45e1b46f1a8964d79ca78d027f3f541603447379206.png)

这里以我们集群的 iptables 规则为例介绍数据包的迁移规则:

PREROUTING 和 OUTPUT 两个检查点: 即入链和出链都会路由到 KUBE-SERVICE 这个全局链中

! [kube-proxy 的三种代理模式 4](https://res-static.hc-cdn.cn/fms/img/a280c29d205516cc9d4f86971b03fc5f1603447379206.png)

接着我们可以看下这个全局链的路由过程, 这里只筛选部分规则说明:

1,-A KUBE-SERVICES -d 10.96.0.1/32 -p tcp -m comment --comment "default/kubernetes: https cluster IP" -m tcp --dport 443 -j KUBE-SVC-NPX46M4PTMTKRN6Y

2,-A KUBE-SVC-NPX46M4PTMTKRN6Y -m comment --comment "default/kubernetes: https" -m recent --rcheck --seconds 10800 --reap --name KUBE-SEP-VFX5D4HWHAGOXYCD --mask 255.255.255.255 --rsource -j KUBE-SEP-VFX5D4HWHAGOXYCD

3,-A KUBE-SEP-VFX5D4HWHAGOXYCD -p tcp -m comment --comment "default/kubernetes: https" -m recent --set --name KUBE-SEP-VFX5D4HWHAGOXYCD --mask 255.255.255.255 --rsource -m tcp -j DNAT --to-destination 172.22.44.62:6443

10.96.0.1 为 clusterip, 发往目的地为 clusterip 的数据包会被转发由 KUBE-SVC-NPX46M4PTMTKRN6Y 规则处理, 接着我们依次找下去, 可以看到最后路由到了 172.22.44.62:6443, 这就是后端的 pod 服务的对应 ip: port.

不同于 userspace, iptables 由 kube-proxy 动态的管理, kube-proxy 不再负责转发, 数据包的走向完全由 iptables 规则决定, 这样的过程不存在内核态到用户态的切换, 效率明显会高很多. 但是随着 service 的增加, iptables 规则会不断增加, 导致内核十分繁忙 (等于在读一张很大的没建索引的表).

! [kube-proxy 的三种代理模式 5](https://res-static.hc-cdn.cn/fms/img/3342f855759f87b227bee56721a058581603447379207.png)

测试环境 80 多个 service 就有 1601 条了, 而且基本都还是单 pod.

# 3, IPVS

! [kube-proxy 的三种代理模式 6](https://res-static.hc-cdn.cn/fms/img/4f9ff62214a7085f7f4e68fff4254d3f1603447379207.png)

一张图说明, 用 ipset 存储 iptables 规则, 这样规则的数量就能够得到有效控制, 而在查找时就类似 hash 表的查找.

相关文章

*   [一篇文章带你详解 HTTP 协议 (网络协议篇一)](http://www.huaweicloud.com/articles/736eef13dc3e4d73b10d155557bb93c5.html " 一篇文章带你详解 HTTP 协议 (网络协议篇一)")
*   [SIP 语音环境中十大经典问题及解决办法](http://www.huaweicloud.com/articles/346c79609e11e11294facc07fdbcc434.html "SIP 语音环境中十大经典问题及解决办法 ")
*   [HIDL 详解 -Android10.0 HwBinder 通信原理 (二)](http://www.huaweicloud.com/articles/07fc665de39f4d2cf143c190069981f2.html "HIDL 详解 -Android10.0 HwBinder 通信原理 (二)")
*   [kubespray 部署 k8s version 1.0](http://www.huaweicloud.com/articles/d866af7822d8141f780768673afc9465.html "kubespray 部署 k8s version 1.0")
*   [kubernetes 使用组件具体介绍](http://www.huaweicloud.com/articles/fc0b33b0ae7f5902dd7ae96fca744b32.html "kubernetes 使用组件具体介绍 ")
*   [2020 前端面试专题整理](http://www.huaweicloud.com/articles/fa6cf4578ad10f6961c9488ae42e9699.html "2020 前端面试专题整理 ")

勿删, copyright 占位



[版权声明] 本文为华为云社区用户转载文章, 如果您发现本社区中有涉嫌抄袭的内容, 欢迎发送邮件至: [huaweicloud.bbs@huawei.com](mailto: huaweicloud.bbs@huawei.com) 进行举报, 并提供相关证据, 一经查实, 本社区将立刻删除涉嫌侵权内容.

为您推荐: [论文摘要](http://www.huaweicloud.com/articles/topic_bac8c2f88d362370c40217c44b9bc158.html " 论文摘要 ") [市场合作](http://www.huaweicloud.com/articles/topic_08f4fde13b8bedd109b0cc99bdc12b8f.html " 市场合作 ") [ios 没有问题](http://www.huaweicloud.com/articles/topic_3bc945e0e98997989c8a195086a4bc91.html "ios 没有问题 ") [activex](http://www.huaweicloud.com/articles/topic_7fc3db11272725df7ecce66dfe02bca4.html "activex") [查看 sshd 端口](http://www.huaweicloud.com/articles/topic_d9b4adb3b8004308df624cd47ea75038.html " 查看 sshd 端口 ") [aspx 预编译](http://www.huaweicloud.com/articles/topic_35109838bd7e4032332ef21e93cb76b5.html "aspx 预编译 ") [js 基本数据类型](http://www.huaweicloud.com/articles/topic_07f8819b4018f448086c7b629636f39c.html "js 基本数据类型 ") [[react.js 点滴知识 ]](http://www.huaweicloud.com/articles/topic_78df9a072203722207c9b42c467e6f62.html "[react.js 点滴知识 ]") [c/c++/c#](http://www.huaweicloud.com/articles/topic_f1bb8aac306315bfba0b0059f65f75b6.html "c/c++/c#") [linux 之 sed](http://www.huaweicloud.com/articles/topic_f7d90f78ea077694c00269bff774642f.html "linux 之 sed")

3

收藏

[](javascript: void(0);)

链接复制成功

复制链接到剪贴板

[](http://service.weibo.com/share/share.php? title= kube-proxy 的三种代理模式&url= https://bbs.huaweicloud.com/blog/163478)

分享文章到微博

分享文章到朋友圈

分享

[上一篇: ElasticSerach 单机安装](http://www.huaweicloud.com/articles/59cb9b0cf35ae09e862f55edd2ce6091.html "ElasticSerach 单机安装 ")

[下一篇: java 异常中常见的问题](http://www.huaweicloud.com/articles/f9796ae6512d0e9cd4baa25a9b4cdffc.html "java 异常中常见的问题 ")

您可能感兴趣

*   [spring cache 相关注解介绍 @Cacheable,@CachePut,@CacheEvict](http://www.huaweicloud.com/articles/0cda049b30b10f9a3deffe266c86a904.html "spring cache 相关注解介绍 @Cacheable,@CachePut,@CacheEvict")

    https://www.jianshu.com/p/49fc4065201a https://blog.csdn.net/poorCoder\_/article/details/55258253 https://www.cnblogs.com/fashflying/p/6908028.html spring cache 的使用 缓存某些方法的执行结果 设置好缓存配置之后我们就可以使用 @Cach...

*   [全栈面试题总结 - 十四道大厂](http://www.huaweicloud.com/articles/6f3988c45d741a306b54599ceeae56c6.html " 全栈面试题总结 - 十四道大厂 ")

    防抖, 顾名思义, 防止抖动, 以免把一次事件误认为多次, 敲键盘就是一个每天都会接触到的防抖操作. 想要了解一个概念, 必先了解概念所应用的场景. 在 JS 这个世界中, 有哪些防抖的场景呢 登录, 发短信等按钮避免用户点击太快, 以致于发送了多次请求, 需要防抖 调整浏览器窗口大小时, resize 次数过于频繁, 造成计算过多, 此时需要一次到位, 就用到了防抖 文本编辑器实时保存, 当无任何更改操作一秒后进行保存...

*   [Spring 中 AOP 相关的 API 及源码解析, 原来 AOP 是这样子的](http://www.huaweicloud.com/articles/75b7b35f2178c54df02a9b585be79229.html "Spring 中 AOP 相关的 API 及源码解析, 原来 AOP 是这样子的 ")

    前言 之所以写这么一篇文章主要是因为下篇文章将结束 Spring 启动整个流程的分析, 从解析配置到创建对象再到属性注入最后再将创建好的对象初始化成为一个真正意义上的 Bean. 因为下篇文章会涉及到 AOP, 所以提前单独将 AOP 的相关 API 及源码做一次解读, 这样可以降低阅读源码的障碍, 话不多说, 我们进入正文! 一个使用 API 创建代理的例子 在进入 API 分析前, 我们先通过两个例子体会下如何使用 API 的方...

*   [最新 Vue 底层原理实现概述](http://www.huaweicloud.com/articles/92c1c4f25938da2d6c5ad57df37e6942.html " 最新 Vue 底层原理实现概述 ")

    Vue, React 这样的框架可以说是现在前端的必备技能, 一个刚入门两三个月的前端都是要会 Vue 的, 而且随着 Vue3.0 发布日程的推进, 使用的人群变得多了, 开始想去了解它. Vue 这么受大众接受, 那么大家有没有想过一个问题? Vue, React 这样的框架已经是基本功, 我们有什么办法能运用得比别人厉害呢? 能够独立用 Vue 写一个项目其实只是入了一个门, 在如今技术快速发展的背景下, 要真的作为一个敢说...

*   [2020 年你必须要会的微前端 -(实战篇)](http://www.huaweicloud.com/articles/b7d01d6a98413c51a97765c4c5862995.html "2020 年你必须要会的微前端 -(实战篇)")

    戳蓝字 " 前端优选 ", 关注我们哦! 最近你有没有经常听到一个词, 微前端? 是不是听上去感觉非常地高大上! 然而~ 微前端其实非常地简单, 非常地容易落地, 而且也非常不高大上~ 那么就来一起看看什么是微前端吧: 一. 为什么需要微前端? 这里我们通过 3W(what, why, how) 的方式来讲解什么是微前端: 1.What? 什么是微前端? 微前端就是将不同的功能按照不同的维度拆分成多个子应用. 通过主应用来...

*   [高级程序员知识学习 (Redis 的扩展应用知识 1)](http://www.huaweicloud.com/articles/0159e9fbadd1a4e7e31804c5b26f6aac.html " 高级程序员知识学习 (Redis 的扩展应用知识 1)")

    Redis 的资源: https://github.com/2462612540/Senior\_Architect.git Redis 基础数据结构 Redis 有 5 种基础数据结构, 分别为: string (字符串), list (列表), set (集合), hash (哈希) 和 zset (有序集合). string (字符串): 字符串 string 是 Redis 最简单的数据结构....

*   [Spark Streaming 运行日志 , 任务监控 Web UI , Kafka , Listener 邮件短信通知](http://www.huaweicloud.com/articles/45e1ccc2639529f3b254a5e67d2530a6.html "Spark Streaming 运行日志 , 任务监控 Web UI , Kafka , Listener 邮件短信通知 ")

    任务监控 一, Spark Web UI 对于 Spark Streaming 任务的监控可以直观的通过 Spark Web UI , 该页面包括 Input Rate, Scheduling Delay, Processing Time 等, 但是这种方法运维成本较高, 需要人工不间断的巡视. 这其中包括接受的记录数量, 每一个 batch 内处理的记录数, 处理时间, 以及总共消耗的时间. 在上述参数之中...

*   [模块一: 持久层框架设计实现及 MyBatis 源码分析](http://www.huaweicloud.com/articles/4a7d52ddecb50ab48856ad9fbc1ea8ee.html " 模块一: 持久层框架设计实现及 MyBatis 源码分析 ")

    面试题 (汇总笔记) 阶段一: 开源框架源码剖析 模块一: 持久层框架设计实现及 MyBatis 源码分析 2020/5/25 1. 通常一个 Xml 映射文件, 都会写一个 Dao 接口与之对应, 请问, 这个 Dao 接口的工作原理是什么? Dao 接口里的方法, 参数不同时, 方法能重载吗? Dao 接口, 就是人们常说的 Mapper 接口, 接口的全限名, 就是映射文件中的 namespace 的值, 接口的方法名, 就是映射文件中 Map...


华为云 40 多款云服务产品 0 元试用活动

[免费套餐, 马上领取!](https://activity.huaweicloud.com/free_test/)

{"data": {"id":"8000-000000437045-0","name":"SEO 专题页栏目分发组 ","type":"1","position":"8000-000000004003-0","status":1,"linkList":\[{"id":"8000-000000661040-0","keyword":"\[华为云•微话题 \] 通讯领域的人工智能项目和算法, 有哪些特殊的挑战? 如何研究解? 参与赢取大号鼠标垫& amp; 双肩包 ","url":"https://bbs.huaweicloud.com/forum/thread-64244-1-1.html","secondDomain": null,"keyTitle": null,"weight":0,"tag": null,"pageTitle": null,"inputType": null,"updateByName":"pWX619094","updateByAccount":"pWX619094","updateAt":"2020-11-18 22:11:37","createByName":"pWX619094","createByAccount":"pWX619094","createAt":"2020-11-18 22:11:37","contentCheckCode":0}, {"id":"8000-000000643914-0","keyword":" 如何 使得识别图像信息 并加以自动运算全程自动化 ","url":"https://bbs.huaweicloud.com/forum/thread-34566-1-1.html","secondDomain": null,"keyTitle": null,"weight":0,"tag": null,"pageTitle": null,"inputType": null,"updateByName":"pWX619094","updateByAccount":"pWX619094","updateAt":"2020-11-18 21:43:58","createByName":"pWX619094","createByAccount":"pWX619094","createAt":"2020-11-18 21:43:58","contentCheckCode":0}, {"id":"8000-000000622948-0","keyword":" 包含头文件 ","url":"https://support.huaweicloud.com/odevg-te-atlas500app/atlasodc\_10\_c30\_0027.html","secondDomain": null,"keyTitle": null,"weight":0,"tag": null,"pageTitle": null,"inputType": null,"updateByName":"pWX619094","updateByAccount":"pWX619094","updateAt":"2020-11-18 21:18:39","createByName":"pWX619094","createByAccount":"pWX619094","createAt":"2020-11-18 21:18:39","contentCheckCode":0}, {"id":"8000-000000606763-0","keyword":" 流程简介 ","url":"https://support.huaweicloud.com/usermanual-obs/obs\_03\_0303.html","secondDomain": null,"keyTitle": null,"weight":0,"tag": null,"pageTitle": null,"inputType": null,"updateByName":"pWX619094","updateByAccount":"pWX619094","updateAt":"2020-11-18 21:02:07","createByName":"pWX619094","createByAccount":"pWX619094","createAt":"2020-11-18 21:02:07","contentCheckCode":0}, {"id":"8000-000000592090-0","keyword":" 简介 ","url":"https://support.huaweicloud.com/bestpractice-evs/evs\_02\_0010.html","secondDomain": null,"keyTitle": null,"weight":0,"tag": null,"pageTitle": null,"inputType": null,"updateByName":"pWX619094","updateByAccount":"pWX619094","updateAt":"2020-11-18 20:59:14","createByName":"pWX619094","createByAccount":"pWX619094","createAt":"2020-11-18 20:59:14","contentCheckCode":0}, {"id":"8000-000000581102-0","keyword":" 终端节点服务管理 ","url":"https://support.huaweicloud.com/usermanual-vpcep/vpcep\_03\_0100.html","secondDomain": null,"keyTitle": null,"weight":0,"tag": null,"pageTitle": null,"inputType": null,"updateByName":"pWX619094","updateByAccount":"pWX619094","updateAt":"2020-11-18 20:52:48","createByName":"pWX619094","createByAccount":"pWX619094","createAt":"2020-11-18 20:52:48","contentCheckCode":0}, {"id":"8000-000000546591-0","keyword":" 如何使用 Shell 脚本来查看多个服务器的端口是否打开?","url":"https://bbs.huaweicloud.com/blogs/175430","secondDomain": null,"keyTitle": null,"weight":0,"tag": null,"pageTitle": null,"inputType": null,"updateByName":"pWX619094","updateByAccount":"pWX619094","updateAt":"2020-11-18 20:01:19","createByName":"pWX619094","createByAccount":"pWX619094","createAt":"2020-11-18 20:01:19","contentCheckCode":0}, {"id":"8000-000000542591-0","keyword":"Zephyr 物联网操作系统初识 (一): 硬件准备与开发环境配置 ","url":"https://bbs.huaweicloud.com/blogs/163505","secondDomain": null,"keyTitle": null,"weight":0,"tag": null,"pageTitle": null,"inputType": null,"updateByName":"pWX619094","updateByAccount":"pWX619094","updateAt":"2020-11-18 19:58:26","createByName":"pWX619094","createByAccount":"pWX619094","createAt":"2020-11-18 19:58:26","contentCheckCode":0}, {"id":"8000-000000518125-0","keyword":"zookeeper 安装 (server 间认证篇 -SASL 配置)","url":"https://bbs.huaweicloud.com/blogs/148833","secondDomain": null,"keyTitle": null,"weight":0,"tag": null,"pageTitle": null,"inputType": null,"updateByName":"pWX619094","updateByAccount":"pWX619094","updateAt":"2020-11-18 17:20:26","createByName":"pWX619094","createByAccount":"pWX619094","createAt":"2020-11-18 17:20:26","contentCheckCode":0}, {"id":"8000-000000493265-0","keyword":"python 基础教程: 包的创建及导入 ","url":"https://bbs.huaweicloud.com/blogs/103366","secondDomain": null,"keyTitle": null,"weight":0,"tag": null,"pageTitle": null,"inputType": null,"updateByName":"pWX619094","updateByAccount":"pWX619094","updateAt":"2020-11-18 15:05:16","createByName":"pWX619094","createByAccount":"pWX619094","createAt":"2020-11-18 15:05:16","contentCheckCode":0}\]},"total": null,"message":"success","status": true}

### 更多推荐

[\[华为云•微话题 \] 通讯领域的人工智能项目和算法, 有哪些特殊的挑战? 如何研究解? 参与赢取大号鼠标垫& amp; 双肩包](https://bbs.huaweicloud.com/forum/thread-64244-1-1.html "[华为云•微话题] 通讯领域的人工智能项目和算法, 有哪些特殊的挑战? 如何研究解? 参与赢取大号鼠标垫& amp; 双肩包 ") [如何 使得识别图像信息 并加以自动运算全程自动化](https://bbs.huaweicloud.com/forum/thread-34566-1-1.html " 如何 使得识别图像信息 并加以自动运算全程自动化 ") [包含头文件](https://support.huaweicloud.com/odevg-te-atlas500app/atlasodc_10_c30_0027.html " 包含头文件 ") [流程简介](https://support.huaweicloud.com/usermanual-obs/obs_03_0303.html " 流程简介 ") [简介](https://support.huaweicloud.com/bestpractice-evs/evs_02_0010.html " 简介 ") [终端节点服务管理](https://support.huaweicloud.com/usermanual-vpcep/vpcep_03_0100.html " 终端节点服务管理 ") [如何使用 Shell 脚本来查看多个服务器的端口是否打开?](https://bbs.huaweicloud.com/blogs/175430 " 如何使用 Shell 脚本来查看多个服务器的端口是否打开?") [Zephyr 物联网操作系统初识 (一): 硬件准备与开发环境配置](https://bbs.huaweicloud.com/blogs/163505 "Zephyr 物联网操作系统初识 (一): 硬件准备与开发环境配置 ") [zookeeper 安装 (server 间认证篇 -SASL 配置)](https://bbs.huaweicloud.com/blogs/148833 "zookeeper 安装 (server 间认证篇 -SASL 配置)") [python 基础教程: 包的创建及导入](https://bbs.huaweicloud.com/blogs/103366 "python 基础教程: 包的创建及导入 ")

### 看了此文的人还看了

[K8S 中 service 的三种工作模式](https://bbs.huaweicloud.com/forum/thread-87944-1-1.html "K8S 中 service 的三种工作模式 ") [[K8S]Service 服务详解, 看这一篇就够了!!](https://bbs.huaweicloud.com/blogs/1638bdaa227341b2a9744bc695830a3c "
[K8S]Service 服务详解, 看这一篇就够了!!
") [Kubernetes 是什么](https://www.huaweicloud.com/zhishi/cce1.html "Kubernetes 是什么 ") [Kubernetes 的 Service 运行原理](https://bbs.huaweicloud.com/blogs/833c8235d9ac49049b0d2de8399e3255 "Kubernetes 的 Service 运行原理 ") [032. 核心组件 -kube-proxy](https://bbs.huaweicloud.com/blogs/863d26ca28f94150aec87dd9b124eb62 "032. 核心组件 -kube-proxy") [灵雀云 k8s 面试题剖析](https://bbs.huaweicloud.com/blogs/87a21877f2d24755855ebc62511776e1 " 灵雀云 k8s 面试题剖析 ") [[Kubernetes 系列] 第 1 篇 架构及组件介绍](https://bbs.huaweicloud.com/blogs/b3440b75d9c711e9b759fa163e330718 "[Kubernetes 系列] 第 1 篇 架构及组件介绍 ") [7\. 丈母娘嫌我不懂 K8s 的 Service 概念, 让我去面壁](https://bbs.huaweicloud.com/blogs/4550581bddab453c87033f2a03cb73bc "7. 丈母娘嫌我不懂 K8s 的 Service 概念, 让我去面壁 ") [Kubernetes 探索实践之网络通信](https://bbs.huaweicloud.com/blogs/0fbec20f81ce4e098bd524fbb06cc7c8 "Kubernetes 探索实践之网络通信 ") [什么是 DDOS 防御](https://www.huaweicloud.com/zhishi/dyl38.html " 什么是 DDOS 防御 ") [kubernetes 安装 kube-router](https://bbs.huaweicloud.com/blogs/efaf56d233a74d739dd85cda76ff6507 "kubernetes 安装 kube-router") [K8s 中的 external-traffic-policy 是什么?](https://bbs.huaweicloud.com/blogs/bf08d57463284ead9a8a53701d54cc05 "K8s 中的 external-traffic-policy 是什么?") [k8s(00) 入门知识介绍](https://bbs.huaweicloud.com/blogs/459615f1e38d4ee4a27351911ab73e70 "k8s(00) 入门知识介绍 ") [[独家]K8S 漏洞报告 | CVE-2019-1002101 解读](https://bbs.huaweicloud.com/forum/thread-16816-1-1.html "[独家]K8S 漏洞报告 | CVE-2019-1002101 解读 ") [Kubernetes(K8S v1.1 版本) 集群管理 Docker 容器之部署篇](https://bbs.huaweicloud.com/blogs/4292cf0cb2454a0bb0f7cb81eaf7c09b "Kubernetes(K8S v1.1 版本) 集群管理 Docker 容器之部署篇 ") [不懂 Kubernetes, 被老板邀请爬山!](https://bbs.huaweicloud.com/blogs/4b41a23ae66b4f488adc31bc490b4715 " 不懂 Kubernetes, 被老板邀请爬山!") [K8s 中的 external-traffic-policy 是什么?](https://bbs.huaweicloud.com/blogs/2712384c167045acbb311171a82a4465 "K8s 中的 external-traffic-policy 是什么?") [使用 NodeLocal DNS Cache](https://bbs.huaweicloud.com/blogs/126f8670af6949f58228ff4af89e3f86 " 使用 NodeLocal DNS Cache") [Kubernetes+ Prometheus+ Grafana 部署笔记](https://bbs.huaweicloud.com/blogs/23b237e5180c4059a14074388d130e89 "Kubernetes+ Prometheus+ Grafana 部署笔记 ") [第七章 九析带你轻松完爆 service mesh - istio 注入](https://bbs.huaweicloud.com/blogs/cb40e1130ffd4b72bb8be84e6cc9c8b4 " 第七章 九析带你轻松完爆 service mesh - istio 注入 ")
