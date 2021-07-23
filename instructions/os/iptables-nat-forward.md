---
title: iptables-nat-forwar
date: 2021-03-27 11:00:00
tags: '操作系统'
categories:
  - ['使用说明', '操作系统']
permalink: iptables-nat-forwar
---

https://blog.csdn.net/qq_39377418/article/details/105102420

# iptables 实现 nat 方式的流量转发

! [](https://csdnimg.cn/release/blogv2/dist/pc/img/reprint.png)

[漫天丶飞雪](https://blog.csdn.net/qq_39377418) 2020-03-25 19:22:23 ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png) 1573 ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png) ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollectionActive.png) 收藏

分类专栏: [Linux](https://blog.csdn.net/qq_39377418/category_8869713.html)

原文链接: [https://www.xuepaijie.com/html/technology/devops/709.html](https://www.xuepaijie.com/html/technology/devops/709.html)

版权

NAT 可以方便的完成这种流量穿通功能, 即把外网数据通过 NAT(中转设备) 来穿透进内网, 内网数据通过 NAT(中转设备) 穿透出外网.

那 linux 下 iptables 如何实现 nat 转发? 这里将以 Debian7 主机下的测试为例.

1, 开启 IP\_FORWARD

<table border="0" cellpadding="0" cellspacing="0"> <tbody> <tr> <td> <p>1</p> <p>2</p> <p>3</p> </td> <td> <p> <code> vi /etc/sysctl.conf</code> </p> <p> <code>#在文件末添加以下一行 (如已有则不必添加, 直接将注释解除即可) </code> </p> <p> <code> net.ipv4.ip_forward=1</code> </p> </td> </tr> </tbody> </table>

2, 使用 IPTABLES, 转发 TCP, UDP 流量

<table border="0" cellpadding="0" cellspacing="0"> <tbody> <tr> <td> <p>1</p> <p>2</p> <p>3</p> <p>4</p> <p>5</p> <p>6</p> <p>7</p> <p>8</p> <p>9</p> </td> <td> <p> <code>#TCP</code> </p> <p> <code> iptables -t nat -A PREROUTING -p tcp --dport 映射地址 -j DNAT --to-destination 目标地址: 目标端口 </code> </p> <p> <code> iptables -t nat -A POSTROUTING -p tcp -d 目标地址 --dport 目标端口 -j SNAT --to-source 映射地址 </code> </p> <p>  </p> <p> <code>#UDP</code> </p> <p> <code> iptables -t nat -A PREROUTING -p udp --dport 映射地址 -j DNAT --to-destination 目标地址: 目标端口 </code> </p> <p> <code> iptables -t nat -A POSTROUTING -p udp -d 目标地址 --dport 目标端口 -j SNAT --to-source 映射地址 </code> </p> <p>  </p> <p> <code>#其中 " 目标地址: 目标端口 " 是目标服务器的 IP 与端口," 映射地址 " 是本机的公网 IP.</code> </p> </td> </tr> </tbody> </table>

3, 保存 IPTABLES

<table border="0" cellpadding="0" cellspacing="0"> <tbody> <tr> <td> <p>1</p> <p>2</p> </td> <td> <p> <code>#这里使用 iptables-save 保存 iptables 配置, 也可以使用其他方法保存.</code> </p> <p> <code> iptables-save > /etc/iptables.up.rules</code> </p> </td> </tr> </tbody> </table>

需要注意的是, 并非所有服务器环境都支持 UDP 转发, 具体操作过程中需要根据实际情况.
