---
title: vps-test
date: 2021-03-07 12:00:00
tags: 'vi'
categories:
  - ['使用说明', '软件']
permalink: vps-test
---

https://www.blueskyxn.com/202006/1188.html

1, LemonBench 全能测试

参考脚本代码

curl -fsL https://ilemonra.in/LemonBenchIntl | bash -s fast

curl -fsSL https://ilemonrain.com/download/shell/LemonBench.sh | bash

wget -qO- https://ilemonrain.com/download/shell/LemonBench.sh | bash

curl -fsSL https://ilemonrain.com/download/shell/LemonBench.sh | bash -s fast

curl -fsSL https://ilemonrain.com/download/shell/LemonBench.sh | bash -s full

wget -qO- https://ilemonrain.com/download/shell/LemonBench.sh | bash -s fast

wget -qO- https://ilemonrain.com/download/shell/LemonBench.sh | bash -s full

2, 内存检测脚本

#CentOS / RHEL

yum install wget -y

yum groupinstall "Development Tools" -y

wget https://raw.githubusercontent.com/FunctionClub/Memtester/master/memtester.cpp

gcc -l stdc++ memtester.cpp

./a.out

#Ubuntu / Debian

apt-get update

apt-get install wget build-essential -y

wget https://raw.githubusercontent.com/FunctionClub/Memtester/master/memtester.cpp

gcc -l stdc++ memtester.cpp

./a.out

3, UnixBench 性能测试

wget --no-check-certificate https://github.com/teddysun/across/raw/master/unixbench.sh

chmod + x unixbench.sh

./unixbench.sh

4,24 小时监测 VPS 延迟

#Debian / Ubuntu

apt-get update

apt-get install python wget screen -y

#CentOS / RHEL

yum install screen wget python -y

安装完运行:

screen -S uping

wget -N --no-check-certificate https://raw.githubusercontent.com/FunctionClub/uPing/master/uping.py

python uping.py

5, ZBench(系统信息 + io+ 网速 + 延迟)

wget -N --no-check-certificate https://raw.githubusercontent.com/FunctionClub/ZBench/master/ZBench-CN.sh && bash ZBench-CN.sh

6, 三网 speedtest

bash <(curl -Lso- https://git.io/superspeed)

7, Bench.sh

秋水逸冰大佬的写的 Bench.sh 脚本

特点

*   显示当前测试的各种系统信息;
*   取自世界多处的知名数据中心的测试点, 下载测试比较全面;
*   支持 IPv6 下载测速;
*   IO 测试三次, 并显示平均值.

wget -qO- bench.sh | bash
#或者
curl -Lso- bench.sh | bash
#或者
wget -qO- 86.re/bench.sh | bash
#或者
curl -so- 86.re/bench.sh | bash

8, SuperBench.sh

老鬼大佬的 SuperBench 测试脚本

### 特点

*   改进了显示的模式, 基本参数添加了颜色, 方面区分与查找.
*   I/O 测试, 更改了原来默认的测试的内容, 采用小文件, 中等文件, 大文件, 分别测试 IO 性能, 然后取平均值.
*   速度测试替换成了 Superspeed 里面的测试, 第一个默认节点是, Speedtest 默认, 其他分别测试到中国电信, 联通, 移动, 各三个不同地区的速度.

wget -qO- --no-check-certificate https://raw.githubusercontent.com/oooldking/script/master/superbench.sh | bash
#或者
curl -Lso- -no-check-certificate https://raw.githubusercontent.com/oooldking/script/master/superb

9, 其他

https://zhuanlan.zhihu.com/p/117547388

点击数:66

### Related posts:

1.  [VPS 测试测试评测脚本合集](https://www.blueskyxn.com/202004/593.html "VPS 测试测试评测脚本合集 ")
2.  [自用 DD 脚本分享, 亲测可用!](https://www.blueskyxn.com/202007/1515.html " 自用 DD 脚本分享, 亲测可用!")
3.  [自用快速装机脚本大全](https://www.blueskyxn.com/202007/1653.html " 自用快速装机脚本大全 ")
4.  [宝塔 Linux 面板 - 开心破解版安装脚本](https://www.blueskyxn.com/202007/1740.html " 宝塔 Linux 面板 - 开心破解版安装脚本 ")
5.  [中国大陆服务器无法访问 Github 素材库 (raw.githubusercontent.com) 的解决方案](https://www.blueskyxn.com/202101/3449.html " 中国大陆服务器无法访问 Github 素材库 (raw.githubusercontent.com) 的解决方案 ")
6.  [原神 1.2 版本爆料汇总 雪山版本 新版本 1129 更新](https://www.blueskyxn.com/202011/2712.html " 原神 1.2 版本爆料汇总 雪山版本 新版本 1129 更新 ")

[

](https://www.hostmsu.ru/aff.php? aff=45 "Starack")

[

](https://www.qexw.com/aff.php? aff=1149 " 企鹅小屋 ")

赏

*   ! [](data: image/gif; base64, R0lGODdhAQABAPAAAMPDwwAAACwAAAAAAQABAAACAkQBADs=)
*   ! [](data: image/gif; base64, R0lGODdhAQABAPAAAMPDwwAAACwAAAAAAQABAAACAkQBADs=)

[Python](https://www.blueskyxn.com/tag/python/) [脚本](https://www.blueskyxn.com/tag/%e8%84%9a%e6%9c%ac/)

[

Previous Post

### 服务器维护日记 20200609

* * *



](https://www.blueskyxn.com/202006/1181.html)

[

Next Post

### 表情包系列: SKYのsticker(v3.1)

* * *



](https://www.blueskyxn.com/202006/1197.html)
