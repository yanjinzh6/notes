---
title: apt-proxy
date: 2021-03-06 15:00:00
tags: 'Linux'
categories:
  - ['使用说明', '操作系统']
permalink: apt-proxy
---

https://blog.csdn.net/allway2/article/details/107546777

# 在 Debian 10 Buster 上配置 APT 代理

! [](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[allway2](https://blog.csdn.net/allway2) 2020-07-23 20:15:06 ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png) 1304 ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png) ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollectionActive.png) 收藏

版权声明: 本文为博主原创文章, 遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议, 转载请附上原文出处链接和本声明.

本文链接: [https://blog.csdn.net/allway2/article/details/107546777](https://blog.csdn.net/allway2/article/details/107546777)

版权

要在 Debian 10 Buster 上配置 APT 代理, 如果您的代理服务器完全支持身份验证, 则需要具有代理服务器的 IP 地址和端口以及身份验证用户名和密码. 可以将 APT 临时或永久配置为使用代理.

临时 APT 代理配置

临时 APT 代理配置涉及创建代理环境变量, 可以是**http\_proxy**或**https\_proxy**, 如下所示;

在我的环境中,**192.168.43.1**是代理服务器 IP,**3128**是代理服务器端口.

对于 HTTP 代理, 只需运行以下命令即可创建一个**http\_proxy**环境变量, 该变量定义了代理服务器和端口.

export **http\_proxy**\='**http**://192.168.43.1:3128'

对于 HTTPS 代理;

export **https\_proxy**\='**https**://192.168.43.1:3128'

如果您的代理服务器支持身份验证, 并且需要用户名 / 密码登录, 则只需使用;

对于 HTTP(S) 代理;

export **http\_proxy**\='**http**://**USERNAME: PASSWORD**@192.168.43.1:3128'

export **https\_proxy**\='**https**://**USERNAME: PASSWORD**@192.168.43.1:3128'

您还可以为 apt 命令加上代理设置前缀, 如下所示;

sudo 'http\_proxy= http://192.168.43.100:3128' apt update

or

sudo 'http\_proxy= http://Username: Password@192.168.43.100:3128' apt update

永久性 APT 代理配置

您可以在 APT 配置文件上永久配置 APT 代理. 例如, 您可以在**/apt/apt.conf.d**目录下创建一个代理配置文件, 如下所示;

vim /etc/apt/apt.conf.d/02proxy

对于 HTTP 代理;

Acquire:: http:: Proxy "http://PROXYSERVERIP: PROXYPORT";

对于 HTTPS;

Acquire::**https**:: Proxy "**https**://PROXYSERVERIP: PROXYPORT";

同时配置 HTTP 和 HTTPS;

Acquire:: http:: Proxy "http://PROXYSERVERIP: PROXYPORT";

Acquire::**https**:: Proxy "**https**://PROXYSERVERIP: PROXYPORT";

或者简单地说;

Acquire {

 HTTP:: proxy "http://PROXYSERVERIP: PROXYPORT";

 HTTPS:: proxy "http://PROXYSERVERIP: PROXYPORT";

}

如果您的代理服务器支持身份验证, 请更换;

http://PROXYSERVERIP: PROXYPORT

与;

http://USERNAME: PASSWORD@PROXYSERVERIP: PROXYPORT

这样您的线条看起来像;

Acquire:: http:: proxy "http://USERNAME: PASSWORD@PROXYSERVERIP: PROXYPORT";

Acquire:: https:: proxy "https://USERNAME: PASSWORD@PROXYSERVERIP: PROXYPORT";

您还可以通过设置   和   变量来配置适用于登录到**/etc/profile.d**上的系统的任何用户的系统范围代理. 例如, 使用以下环境变量创建一个文件**/etc/profile.d/proxy.sh**.http\_proxyhttps\_proxy

vim /etc/profile.d/proxy.sh

export http\_proxy='http://USERNAME: PASSWORD@192.168.43.1:3128'

export https\_proxy='https://USERNAME: PASSWORD@192.168.43.1:3128'

source 代理配置文件以重新加载环境变量.

source /etc/profile.d/proxy.sh

如果您使用的是 bash shell, 则要设置适用于单个用户的代理, 可以编辑**$ HOME / .bashrc**文件并添加以下行:

vim $HOME/.bashrc

export http\_proxy='http://USERNAME: PASSWORD@192.168.43.1:3128'

export https\_proxy='https://USERNAME: PASSWORD@192.168.43.1:3128'

源$ HOME / .bashrc 文件:

source $HOME/.bashrc

现在, 如果您尝试运行 apt 命令, 您将注意到它尝试连接到代理服务器. 如果连接成功, 则您的 APT 将正常运行.

apt update

0% \[Connecting to 192.168.43.1 (192.168.43.1)\]...

这就是如何在 Debian 10 Buster 上配置 APT 代理的全部内容.
