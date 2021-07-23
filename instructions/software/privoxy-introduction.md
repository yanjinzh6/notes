---
title: privoxy 使用
date: 2021-01-30 12:00:00
tags: 'privoxy'
categories:
  - ['使用说明', '软件']
permalink: privoxy-introduction
---

## 简介

Privoxy 是一个 代理软件

简单说, 就是进出你电脑的流量守门人. 借由 Privoxy, 我们可以控制出去的请求, 还可以改写返回的响应. 不必要的请求 – 比如视频广告的地址, 图片广告的地址, 我们可以直接 `block` 掉; 不必要的响应内容 – 比如页面中的文字广告, 我们可以借由 `filter` 过滤掉, 不让它们在浏览器页面上显示.

## 安装 Privoxy

[Privoxy 支持的平台](https://www.privoxy.org/faq/installation.html#WHICHOS) 非常多:

针对各个平台, Privoxy 均提供有 [安装说明](https://www.privoxy.org/user-manual/installation.html#INSTALLATION-PACK-WIN).

下面简单提一下几个平台.

1.  Windows 平台

    Windows 平台下的安装通常都可视化, 下载安装包, 双击, 然后根据提示, 一路点击即可安装.

2.  Linux 平台

    多数时候可以通过仓库安装.

    比如 Ubuntu:

    ```
    sudo apt-get install privoxy
    ```

    又比如 openSUSE:

    ```
    sudo zypper install privoxy
    ```
3.  mac 平台

    mac 上如果有安装 homebrew, 则可以执行 `brew install privoxy` 来安装 Privoxy.


实在喜欢折腾的, 就下载 [源代码](http://sourceforge.net/projects/ijbswa/files/Sources/) 自己编译安装.

## 代理转发

### 配置文件说明

### 主配置文件

  对于 Windows 系统, 主配置文件放置在跟 privoxy.exe 相同的目录下, 文件名为 config.txt

  对于 "Linux/Unix", 主配置文件放置在 /etc/privoxy 目录下, 文件名为 config

  Privoxy 的配置文件, 都是纯文本文件. 如果某一行是以 井号 # 开头, 说明这一行是注释.

### 如何定制 Privoxy 的 " 监听端口及绑定地址 "

  先来说最最基本的配置——设定 Privoxy 的监听端口号和绑定的地址.

  介绍这个的目的, 是让你先稍微熟悉一下——如何修改 Privoxy 的主配置文件. 因为后面的定制, 需要经常去修改它.

  Privoxy 的监听端口号, 默认是 8118, 默认绑定的地址是 127.0.0.1(这个地址代表 " 当前系统 "). 由于默认是绑定在 127.0.0.1 这个地址, 所以只有当前系统的软件才可以连接到 Privoxy 的监听端口.

  如果你希望其它操作系统的软件也可以连接到 Privoxy 的监听端口, 可以修改绑定的地址, 把 127.0.0.1 改为 0.0.0.0 表示绑定在 " 任意地址 ".

  操作步骤如下:

  找到 " 主配置文件 ", 用你比较顺手的文本编辑器打开, 在尾部增加如下一行

```
listen-address  0.0.0.0:8118
```

  如果你不喜欢 8118 这个端口号, 也可以换成别的.

  修改完之后, 启动 Privoxy, 然后在命令行使用 netstat 命令, 就可以看到多出来一个 8118 的端口.

### 如何定制 "HTTP 代理转发 "

  "HTTP 代理转发 " 就是说: Privoxy 把自己收到的 HTTP 请求转给另一个 HTTP 代理, 再由该代理转到你最终访问的网站.

  HTTP 代理转发的语法如下 (把该语法添加在 " 主配置文件 " 尾部):

```
forward  target_pattern  http_proxy: port
```

  语法解释:

  该命令分 3 段, 各段之间用空格分开 (可以用单个空格, 也可以多个空格)

  第 1 段的 forward 是固定的, 表示: 这是 HTTP 转发

  第 2 段的 target\_pattern 是个变量, 表示: 这次转发只针对特定模式的 HTTP 访问目标

  第 3 段的 http\_proxy: port 也是变量, 表示: 要转发给某个 HTTP 代理 (IP 冒号 端口). 如果 " 第 3 段 " 只写一个单独的小数点, 表示直连 (不走代理).

举例 1

```
forward  /  127.0.0.1:8080
```

上述这句表示:

把所有的 HTTP 请求都转发给本机 (127.0.0.1 表示本机) 的 8080 端口

**target\_pattern 设置为 " 单个斜线 ", 表示匹配 " 所有的 URL 网址 "**

举例 2

```
forward  .google.com/  127.0.0.1:8080
```

上述这句表示:

如果 HTTP 请求是发送给 .google.com 这个域名的下级域名, 那么就把该 HTTP 请求转发给本机 (127.0.0.1 表示本机) 的 8080 端口

.google.com 的写法, 可以匹配如下这类域名:

www.google.com

mail.google.com

plus.google.com

(以此类推)

补充说明:

本例子中, target\_pattern 变量是 .google.com/

这个写法表示: 该变量只匹配域名, 不匹配网页的路径. 根据 Privoxy 的语法规则, 最末尾的斜线可以省略.

因此, 本例子也可以等价地写为:

```
forward  .google.com  127.0.0.1:8080
```

举例 3

```
forward-socks5 .onion localhost:9050 .
```

上述这句表示:

把顶级域名为 .onion 的 HTTP 请求都转发给本机 (localhost 也可以用来表示本机, 相当于 127.0.0.1) 的 TOR SOCKS 端口

补充说明:

普通互联网的域名, 顶级域名不会是 .onion

这个顶级域名, 是专门用于 TOR 暗网的. 因此, 本例子就是用来转发 TOR 暗网的 HTTP 请求, 让 Privoxy 把这类请求转给 TOR 的 SOCKS 端口.

### 如何定制 "SOCKS 代理转发 "

  所谓的 "SOCKS 代理转发 ", 就是说: Privoxy 把自己收到的 HTTP 请求转给另一个 SOCKS 代理; 如果需要的话, 还可以由这个 SOCKS 代理再转给另一个 HTTP 代理.

示意图 1

(先转发到 SOCKS 代理, 然后转到目标站)

```
 |====＞|       |====＞|       |====＞|
浏览器 |     |Ｐｒｉｖｏｘｙ|     |ＳＯＣＫＳ代理 |     | 目标网站
   |＜==== |       |＜==== |       |＜==== |
```

   | 此阶段是 |       | 此阶段是 |       | 此阶段是 |

   |ＨＴＴＰ |       |ＳＯＣＫＳ|       |ＨＴＴＰ |

示意图 2

(先转发到 SOCKS 代理, 再转发到某个 HTTP 代理, 最后才转到目标站)

```
 |====＞|       |====＞|       |====＞|      |====＞|
浏览器 |     |Ｐｒｉｖｏｘｙ|     |ＳＯＣＫＳ代理 |     |ＨＴＴＰ代理 |     | 目标网站
   |＜==== |       |＜==== |       |＜==== |      |＜==== |
```

   | 此阶段是 |       | 此阶段是 |       | 此阶段是 |      | 此阶段是 |

   |ＨＴＴＰ |       |ＳＯＣＫＳ|       |ＨＴＴＰ |      |ＨＴＴＰ |

  SOCKS 代理转发, 包括如下几种语法:

```
forward-socks4   target_pattern  socks_proxy: port  http_proxy: port
forward-socks4a  target_pattern  socks_proxy: port  http_proxy: port
forward-socks5   target_pattern  socks_proxy: port  http_proxy: port
forward-socks5t  target_pattern  socks_proxy: port  http_proxy: port
```

  语法解释:

  该命令分 4 段, 各段之间用空格分开 (可以用单个空格, 也可以多个空格)

  第 1 段是以 forward 开头的, 表示 SOCKS 转发的类型. 目前支持 4 种类型.

前面 3 种 (forward-socks4 forward-socks4a forward-socks5) 分别对应不同版本的 SOCKS 协议.

最后一种 forward-socks5t 比较特殊, 是基于 SOCKS5 协议版本, 但是加入针对 TOR 的扩展支持 (优化了性能). 只有转发给 TOR 的 SOCKS 代理, 才需要用这个类型.

  第 2 段的 target\_pattern 是个变量, 表示: 这次转发只针对特定模式的 HTTP 访问目标

  第 3 段的 socks\_proxy: port 也是变量, 表示: 要转发给某个 SOCKS 代理 (IP 冒号 端口)

  第 4 段的 http\_proxy: port 也是变量, 表示: 在经由前面的 SOCKS 代理之后, 再转发给某个 HTTP 代理 (IP 冒号 端口)

**第 4 段如果写成一个单独的小数点, 相当于 " 示意图 1"; 如果第 4 段填写了具体的 IP 和端口, 相当于 " 示意图 2".**

举例 1

  如果你本机安装了 TOR Browser 软件包, 可以使用如下语法, 把 Privoxy 收到的 HTTP 请求转发给 TOR Browser 内置的 SOCKS 代理.

```
forward-socks5  /  127.0.0.1:9150  .
```

### 根据 " 访问的网站 " 分流到不同的翻墙通道

```
forward  /  .
forward  .youtube.com  127.0.0.1:8000
forward-socks5  program-think.blogspot.com  127.0.0.1:9150  .
```

第 1 行表示: 因为匹配的目标是 / 表示匹配 " 所有的网址 ". 所以这条可以看成是 " 默认规则 ".(**在 Privoxy 里面," 后面的规则 " 会覆盖 " 前面的规则 ", 所以 " 默认规则 " 总是写在开头**).

第 2 行表示: 凡是 .youtube.com 的下级域名都走 127.0.0.1:8000 的 HTTP 代理.

第 3 行表示: 访问 program-think.blogspot.com 站点都走 127.0.0.1:9150 的 SOCKS 代理.(提醒一下: 此行末尾的小数点别漏看了)

例如, 安装 SSCap4.0 以后, 内网不可以访问, 可以修改配置文件:

```
forward-socks5 / 127.0.0.1:1080 .
forward  18.16.*.*  .
```

这样 18.16._._的内网就即可访问.

## 代理切换

如果想把不同的网址切换到不同的代理, 就使用 action 功能, 可以使用 user.action 文件,

或者自定义一个文件, 在 config 中添加下文, 这表示添加一个动作文件, 文件名是 pac.action. 在同目录下建立文件 "pac.action", 并写入配置.

```
actionsfile pac.action
```

示例:

```
{{alias}}
direct      = + forward-override{forward .}
ssh         = + forward-override{forward-socks5 127.0.0.1:7000 .}
gae         = + forward-override{forward 127.0.0.1:8000}
default     = direct
#========== 默认代理 ==========
{default}
/
#========== 直接连接 ==========
{direct}
.edu.cn
202.117.255.
222.24.211.70
#========== SSH 代理 ==========
{ssh}
.launchpad.net
#========== GAE 代理 ==========
{gae}
.webupd8.org
222.24.211.70
```

上面的 {{alias}} 部分定义了一些缩写, 注意 http 代理和 socks 代理的写法不同.

后面的如 {direct} 部分定义对哪些地址应用这个代理. 其中 "/" 表示全部地址. 注意一个 URL 的域名部分只能用 glob 匹配, 而地址部分可以用复杂的正则表达式. 具体可以看 Privoxy 的文档

这些规则在后面的会覆盖前面的, 比如 222.24.211.70 实际是以 gae 代理访问的. 这样可以实现一些稍微复杂的功能

## 参考:

[Privoxy 教程](https://blog.zfanw.com/privoxy-tutorial/)

[如何用 Privoxy 辅助翻墙?](https://program-think.blogspot.com/2014/12/gfw-privoxy.html)

[强大的代理调度器代理 Privoxy](https://www.igfw.net/archives/1178)

[privoxy——广告过滤和自动代理切换](https://www.lainme.com/doku.php/blog/2011/04/privoxy_%E5%B9%BF%E5%91%8A%E8%BF%87%E6%BB%A4%E5%92%8C%E8%87%AA%E5%8A%A8%E4%BB%A3%E7%90%86%E5%88%87%E6%8D%A2)

[polipo/privoxy 实现 Linux 系统全局 / 自动代理](https://juejin.im/post/5c91ff5ee51d4534446edb9a)

[我的上网技巧](https://paper.tuisec.win/detail/e7c62db3dd697b7)







[! [](data: image/png; base64, iVBORw0KGgoAAAANSUhEUgAAADwAAAA8CAYAAAA6/NlyAAAAAXNSR0IArs4c6QAAAERlWElmTU0AKgAAAAgAAYdpAAQAAAABAAAAGgAAAAAAA6ABAAMAAAABAAEAAKACAAQAAAABAAAAPKADAAQAAAABAAAAPAAAAACL3+ lcAAAAV0lEQVRoBe3QMQEAAADCoPVP7WkJiEBhwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGPjAADh8AAFUUgPVAAAAAElFTkSuQmCC)](/user/3737995265976430)

2019 年 04 月 05 日 阅读 12109

关注

! [](data: image/png; base64, iVBORw0KGgoAAAANSUhEUgAAADwAAAA8CAYAAAA6/NlyAAAAAXNSR0IArs4c6QAAAERlWElmTU0AKgAAAAgAAYdpAAQAAAABAAAAGgAAAAAAA6ABAAMAAAABAAEAAKACAAQAAAABAAAAPKADAAQAAAABAAAAPAAAAACL3+ lcAAAAV0lEQVRoBe3QMQEAAADCoPVP7WkJiEBhwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGPjAADh8AAFUUgPVAAAAAElFTkSuQmCC)

# polipo/privoxy 实现 Linux 系统全局 / 自动代理

## 前言

操作系统为 Ubuntu.
客户端代理软件为 Python 版本 Shadow 和谐 socks 自带的 sslocal.

> SS 安装和配置过程不再赘述. 默认本地端口 `1080`, 这里改成了 `1081`.

sslocal 是 socks5 代理, 需要一个软件进行 socks5 和 HTTP 的转换.
下面介绍 **polipo** 和 **privoxy** 两种.

polipo 貌似只能全局代理, privoxy 全局 / 自动两种代理方式都可以实现.

> 全局代理下, 访问 `localhost` 时也会走代理, 可能导致无法正常访问本地服务.

## polipo 实现全局代理

安装 polipo:

```
apt-get update
apt-get install polipo
复制代码
```

polipo 的配置文件 `/etc/polipo/config` 初始内容只有 `logSyslog` 和 `logFile` 两项.
添加以下内容:

```
# SS 的代理地址
socksParentProxy = "127.0.0.1:1081"
# 类型
socksProxyType = socks5
# 转换为 HTTP 之后的端口
proxyPort = 8123
# 下面的就不清楚了
chunkHighMark = 50331648
objectHighMark = 16384
serverMaxSlots = 64
serverSlots = 16
serverSlots1 = 32
proxyAddress = "0.0.0.0"
复制代码
```

`8123` 就是 HTTP 代理的端口了.

接下来把代理地址添加到环境变量. 在 `/etc/profile` 添加以下内容:

```
export http_proxy="http://127.0.0.1:8123"
export https_proxy="http://127.0.0.1:8123"
复制代码
```

重新载入:

```
source /etc/profile
复制代码
```

启动 polipo:

```
service polipo start
复制代码
```

测试一下:

```
curl www.google.com
复制代码
```

## privoxy 实现全局和自动代理

privoxy 可以配置 .action 格式的代理规则文件. 通过控制规则文件实现全局和自动代理.

action 文件可以手动编辑, 也可以从 [gfwlist](https://github.com/gfwlist/gfwlist) 生成.
下面将先介绍 privoxy 的安装配置, 再介绍 action 文件的生成.

### 安装配置

安装 privoxy:

```
apt-get update
apt-get install privoxy
复制代码
```

进入目录 `/etc/privoxy`, 可以看到目录结构大致为:

*   `config` 配置文件, 这个文件很长..
*   `*.action` 代理规则文件
*   `*.filter` 过滤规则文件
*   `trust` 不造干嘛用
*   `templates/` 同上

开始修改配置文件.

privoxy 有 filter (过滤) 的功能, 可以用来实现广告拦截. 不过这里只希望实现自动代理, 在配置文件中把 filter 部分注释掉:

```
# 大约在 435 行
# filterfile default.filter
# filterfile user.filter      # User customizations
复制代码
```

我们将使用自定义的 action 文件, 所以把默认的 action 文件注释掉, 并添加自定义文件:

```
# 386 行左右
# 默认的 action 文件
# actionsfile match-all.action # Actions that are applied to all sites and maybe overruled later on.
# actionsfile default.action   # Main actions file
# actionsfile user.action      # User customizations
# 自定义 action 文件
actionsfile my.action
复制代码
```

可以指定转换后的 HTTP 代理地址, 这里直接使用默认端口 `8118`:

```
# 785 行左右
listen-address  127.0.0.1:8118
listen-address  [::1]:8118
复制代码
```

如果代理规则直接写在配置文件 `config` 中, 那么代理规则和本地 SS 代理地址是写在一起的:

```
# / 代表匹配全部 URL, 即全局代理
forward-socks5 / 127.0.0.1:1081 .
复制代码
```

或

```
# 根据规则自动代理
forward-socks5 .google.com 127.0.0.1:1081 .
复制代码
```

**注意! 每行最后还有一个点.**

实现全局代理就是第一种写法了.

但是如果要自动代理, 第二种直接写在配置文件里的做法其实不太合适, 更合适的做法是写成 action 文件, 配置文件中只管引用.

把上面的注释掉.
新建 action 文件 `my.action`, 内容如下:

```
# 这一行表示本 action 文件中所有条目都使用代理
{+ forward-override{forward-socks5 127.0.0.1:1081 .}}
# 添加一条规则
.google.com
复制代码
```

把 privoxy 转换后的地址 `http://127.0.0.1:8118` 添加到环境变量, 可以参照 polipo 部分.

启动 privoxy, 这时应该可以正常访问 Google 了:

```
service privoxy start
curl www.google.com
复制代码
```

下面看一下怎么用 gfwlist 生成 action 文件.

### 生成 action 文件

> 配置文件 `config` 或 action 文件修改后不需要重启 privoxy.

使用的工具是 [gfwlist2privoxy](https://github.com/snachx/gfwlist2privoxy). 这个工具很简单, 文档就几行, 写得也很清楚.

安装:

```
pip install gfwlist2privoxy
复制代码
```

gfwlist2privoxy **不支持 python3.x**, 安装时注意使用的是 `pip2` 还是 `pip3`.

参数说明:

*   `-i`/`--input` 输入, 本地 gfwlist 文件或文件 URL. 这里使用上面的 [gfwlist](https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt)
*   `-f`/ `--file` 输出, 即生成的 action 文件的目录. 这里输出到 `/etc/privoxy/gfwlist.action`
*   `-p`/ `--proxy` SS 代理地址, 生成后可以修改. 这里是 `127.0.0.1:1081`
*   `-t`/ `--type` 代理类型, 生成后也可以修改. 这里是 `socks5`
*   `--user-rule` 用户自定义规则文件, 这个文件中的规则会被追加到 gfwlist 生成的规则后面

示例:

```
gfwlist2privoxy -i https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt -f /etc/privoxy/gfwlist.action -p 127.0.0.1:1081 -t socks5
复制代码
```

得到文件 `/etc/privoxy/gfwlist.action`:

```
# gfwlist.action generated by gfwlist2privoxy, 2018-08-02 07:36:00 +0000
# https://github.com/snachx/gfwlist2privoxy

{+ forward-override{forward-socks5 127.0.0.1:1081 .}}

# 规则列表
...
复制代码
```

最后, 把 `/etc/privoxy/config` 中的 `actionsfile my.action` 改为 `actionsfile gfwlist.action` 就完成了.

## 其他

1.  还有一种自动代理的方法使用了 [cow](http://www.nasyun.com/thread-24853-1-1.html), 还没试过.

2.  环境变量的配置 很多教程都只添加了 `http_proxy` 一项, 但是实际使用中发现也需要设置 `https_proxy`.

    另外, 关于地址的写法, 只写 `127.0.0.1:8123` 时, 遇到过有软件不能识别的情况, 改为**写完整的地址** `http://127.0.0.1:8123/` 就不会有问题了.


## 参考链接

[Raspberry PI+ SS+ Polipo 实现科学上网](https://medium.com/@molimowang/raspberry-pi-shadowsocks-polipo%E5%AE%9E%E7%8E%B0%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91-eae1b7eeb779)
[Privoxy 教程](https://blog.zfanw.com/privoxy-tutorial/)
[使用 Privoxy 实现通用选择性代理功能](http://cckpg.blogspot.com/2011/06/privoxy.html)

https://www.cnblogs.com/hongdada/p/10787924.html
https://juejin.cn/post/6844903813393055751
