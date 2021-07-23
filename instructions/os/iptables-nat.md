---
title: iptables-nat
date: 2021-03-14 17:00:00
tags: '操作系统'
categories:
  - ['使用说明', '操作系统']
permalink: iptables-nat
---

https://blog.csdn.net/shyodx/article/details/7977432

# iptables IP 转发

! [](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[shyodx](https://blog.csdn.net/shyodx) 2012-09-14 09:09:31 ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png) 4645 ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png) ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollectionActive.png) 收藏

分类专栏: [iptables](https://blog.csdn.net/shyodx/category_1237577.html) [Linux](https://blog.csdn.net/shyodx/category_912066.html) 文章标签: [服务器](https://www.csdn.net/tags/MtTaEg0sNDcxOTgtYmxvZwO0O0OO0O0O.html)

版权声明: 本文为博主原创文章, 遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议, 转载请附上原文出处链接和本声明.

本文链接: [https://blog.csdn.net/shyodx/article/details/7977432](https://blog.csdn.net/shyodx/article/details/7977432)

版权

有两台服务器:

*   A:219.246.xxx.yyy. 这是一个内网地址, 外网无法访问.
*   B:202.201.aaa.bbb. 这是一个公网地址, 它和 xxx.lzu.edu.cn 这个域名绑定.

两台服务器上都跑了一个 HTTP 的服务, 他们之间可以互相访问. 现在希望访问 mmm.lzu.edu.cn 时, 能自动跳转到 219.246.xxx.yyy 上, 因此可以做 IP 转发, 或叫 IP 映射. 来使所有发到 202.201.aaa.bbb 的请求包都通过它转发到 219.246.xxx.yyy 上. 而所有应答的数据包, 又能从 219.246.xxx.yyy 通过 202.201.aaa.bbb 回到请求者的地方.



```
# iptables -t nat -A PREROUTING -d 202.201.aaa.bbb -j DNAT --to-destination 219.246.xxx.yyy
# iptables -t nat -A POSTROUTING -d 219.246.xxx.yyy -j SNAT --to 202.201.aaa.bbb
```

另外, 查看非默认表的规则需要指定表明:

```
# iptables -t nat -L
```
需要删除某条规则:

```
# iptables -t nat -L --line-number // 列出规则的编号
# iptables -t nat -D PREROUTING 2 // 删除 PREROUTING 中的第 2 条规则
```
