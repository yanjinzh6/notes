---
title: eh-forwarder-bot-tg-msg
date: 2021-03-06 18:00:00
tags: 'tg'
categories:
  - ['使用说明', '软件']
permalink: eh-forwarder-bot-tg-msg
---

https://www.moerats.com/archives/931/

Rat's

使用 EH Forwarder Bot 实现 Telegram 同时收发多个微信 /QQ 消息

说明: EH Forwarder Bot 是一个可扩展的聊天隧道框架, 允许用户一次发送和接收来自多个 IM 平台的消息, 并...

扫描右侧二维码阅读全文

! [](https://www.moerats.com/usr/themes/handsome/libs/GetCode.php? type= url&content= https://www.moerats.com/archives/931/)

19
2019/04

# 使用 EH Forwarder Bot 实现 Telegram 同时收发多个微信 /QQ 消息

*   博主: [Rat's](https://www.moerats.com/author/1/)
*    发布时间:2019 年 04 月 19 日
*    41291 次浏览
*     [122 条评论](#comments)
*    8181 字数
*   分类: [主机教程](https://www.moerats.com/category/zjjc/)

1.   [首页](https://www.moerats.com/ " 返回首页 ")
2.  正文  

分享到: [](http://sns.qzone.qq.com/cgi-bin/qzshare/cgi_qzshare_onekey? url= https://www.moerats.com/archives/931/&title= 使用 EH Forwarder Bot 实现 Telegram 同时收发多个微信 /QQ 消息&site= https://www.moerats.com/) [](http://service.weibo.com/share/share.php? url= https://www.moerats.com/archives/931/&title= 使用 EH Forwarder Bot 实现 Telegram 同时收发多个微信 /QQ 消息)

**说明:**`EH Forwarder Bot` 是一个可扩展的聊天隧道框架, 允许用户一次发送和接收来自多个 `IM` 平台的消息, 并最终远程管理他们的帐户, 目前可以实现的 `Telegram` 收发 `QQ`, 微信,`Facebook Messenger` 等消息, 你也可以同时一起收发 `N` 个微信,`N` 个 `QQ` 等, 这里就说下 `Telegram` 收发微信 /`QQ` 消息的手动安装及 `Docker` 安装.

## 收发微信

```
提示: 使用该功能前, 请先确认所使用的微信能成功登录 WEB 端, 不然后面会报错.
```

**项目地址:**[https://github.com/blueset/ehForwarderBot](https://github.com/blueset/ehForwarderBot)

所使用的模块地址:

```
#Telegram 模块
https://github.com/blueset/efb-telegram-master
#微信模块
https://github.com/blueset/efb-wechat-slave
```

其他模块地址→[传送门](https://github.com/blueset/ehForwarderBot/wiki/Channels-Repository), 包括 `Facebook Messenger` 等模块, 有兴趣的可以看下.

**环境要求:**`Python 3.6+`,`EH Forwarder Bot 2.0+`,`ffmpeg`,`libmagic`,`libwebp`

手动教程适用于 `Debian`,`CentOS`,`Ubuntu`, 如果你想用 `Ubuntu` 的话, 最好使用 `18.04+` 版本.

**1, 安装依赖**

```
#CentOS 系统
yum install file-devel libwebp-tools git screen -y

#Debian/Ubuntu 系统
apt install libwebp-dev libmagic-dev git screen -y
```

**2, 安装 Python3.6**

```
#CentOS 系统
wget https://www.moerats.com/usr/shell/Python3/CentOS_Python3.6.sh && sh CentOS_Python3.6.sh
#Debian 系统
wget https://www.moerats.com/usr/shell/Python3/Debian_Python3.6.sh && sh Debian_Python3.6.sh
#Ubuntu 系统
apt update
apt install python3-pip python3-setuptools python3-dev -y
```

**3, 安装 ffmpeg**

```
#下载 ffmpeg 二进制
wget https://www.moerats.com/usr/down/ffmpeg/ffmpeg-git-$(getconf LONG_BIT) bit-static.tar.xz
#解压文件
tar xvf ffmpeg-git-*-static.tar.xz
#移动 ffmpeg 可执行文件
mv ffmpeg-git-*/ffmpeg  ffmpeg-git-*/ffprobe /usr/bin/
#删除文件
rm -rf ffmpeg-git-*
```

**4, 安装框架**

```
#安装稳定版
pip3 install ehforwarderbot

#安装开发版, 建议安装开发版, bug 修复快些, 功能也新
pip3 install git+ https://github.com/blueset/ehforwarderbot.git
```

**5, 安装 TG 和微信模块**

```
pip3 install efb-telegram-master efb-wechat-slave
```

**6, 启用模块**
先新建配置文件夹和配置文件 `config.yaml`, 使用命令:

```
#default 为配置文件默认的文件夹, 你也可以命名为其它的, 不会玩的就默认
mkdir -p ~/.ehforwarderbot/profiles/default
nano ~/.ehforwarderbot/profiles/default/config.yaml
```

参考代码为:

```
#请根据实际情况修改
master_channel: foo.demo_master
slave_channels:
- foo.demo_slave
- bar.dummy
middlewares:
- foo.null
```

以上对应的均为模块名称, 模块参考→[传送门](https://github.com/blueset/ehForwarderBot/wiki/Channels-Repository), 比如这里博主只用了 `Telegram` 和 `WeChat` 模块, 所以大致配置为:

```
master_channel: blueset.telegram
slave_channels:
- blueset.wechat
```

然后使用 `Ctrl+ x`,`y` 保存退出.

这只是登录一个微信号, 如果你要同时登录多个微信号, 那么配置文件就需要改为:

```
#比如我要同时登录并收发 3 个微信号
master_channel: blueset.telegram
slave_channels:
- blueset.wechat
- blueset.wechat#moe123
- blueset.wechat#rats321
```

只需要在后面使用 `#` 指定一个 `ID` 号, 该 `ID` 号只能有字母, 数字和下划线, 即正则表达式 `[a-zA-Z0-9_]+`, 想登录几个账户就加几个. 如果你使用 `QQ`,`Facebook Messenger` 模块的话, 设置方法也一样.

**7, 建立 TG 配置文件**
建立配置文件前需要先获取 `Telegram` 的 `Token` 和 `Userid`, 获取方法如下:

```
#Telegram 的 Token 获取
1, 在 Telegram 关注@BotFather
2, 再到对话框依次输入:/start=>/newbot, 然后会要你给机器人命名 (如: MoeratsBot), 命名完成会给你一个 Token.

#Telegram 群 Userid 获取
1, 先和你的机器人聊天, 随便发一句话.
2, 在浏览器输入 https://api.telegram.org/botxx: xx/getUpdates(其中 xx: xx 为 Token), 然后 chat 后面的 id 即为你的 userid.
```

再新建一个 `Telegram` 模块配置文件夹和配置文件 `config.yaml`, 使用命令:

```
#同样的也建在 default 文件夹, 如果你上面更改了 default 文件夹, 那这里也要更改
mkdir ~/.ehforwarderbot/profiles/default/blueset.telegram
nano ~/.ehforwarderbot/profiles/default/blueset.telegram/config.yaml
```

填入以下代码:

```
token: "12345: moerats"
admins:
- 765432
```

然后使用 `Ctrl+ x`,`y` 保存退出. 上面所对应的参数分别为 `Token` 和 `Userid`. 关于 `Telegram` 模块的更多玩法可以参考→[传送门](https://github.com/blueset/efb-telegram-master).

**8, 启动**

```
#该命令会默认从 default 文件夹读取信息, 如果你之前建的是 moerats 文件夹, 那命令应该为 ehforwarderbot -p moerats
ehforwarderbot
```

这时候会给一个微信二维码或者二维码链接你, 放到浏览器打开扫描登录即可, 如果你设置了同时登录多个账户, 那设置几个就需要登录几个.

然后使用 `Ctrl+ C` 断开运行, 再使用命令后台运行:

```
screen -dmS EHF ehforwarderbot
```

最后你的微信消息会通过机器人发送给你, 你也可以通过机器人将消息发送给指定好友.

## 收发 QQ 消息

```
提示: 这里随便提了下, 了解下就行了, 建议使用下面 Docker 方式安装.
```

所使用的模块地址:

```
#Telegram 模块
https://github.com/blueset/efb-telegram-master
#QQ 模块
https://github.com/milkice233/efb-qq-slave
```

由于方法写的很大概, 所以需要你把收发微信的方法看懂, 这里 `EH Forwarder Bot` 只支持 ` 酷 Q` 客户端, 一般采用 `Docker` 的方法在 `Linux` 上安装酷 `Q`, 方法很久以前就说过了, 参考→[传送门](https://www.moerats.com/archives/802/), 不过启动命令变了下, 也就是安装 `wine-coolq` 的命令.

安装 `TG` 和 `QQ` 模块:

```
pip3 install efb-telegram-master efb-qq-slave
```

安装 `wine-coolq`:

```
mkdir coolq  #包含 CoolQ 程序文件
docker run -ti --rm --name cqhttp-test --net="host" \
     -v $(pwd)/coolq:/home/user/coolq     `#mount coolq folder` \
     -p 9000:9000                         `#网页 noVNC 端口 ` \
     -p 5700:5700                         `#酷 Q 对外提供的 API 接口的端口 ` \
     -e VNC_PASSWD= MAX8char               `#请修改 VNC 密码!!!!` \
     -e COOLQ_PORT=5700                   `#酷 Q 对外提供的 API 接口的端口 ` \
     -e COOLQ_ACCOUNT=123456              `#在此输入要登录的 QQ 号, 虽然可选但是建议填入 ` \
     -e CQHTTP_POST_URL= http://127.0.0.1:8000   `#efb-qq-slave 监听的端口 / 地址用于接受传入的消息 ` \
     -e CQHTTP_SERVE_DATA_FILES= yes       `#允许以 HTTP 方式访问酷 Q 数据文件 ` \
     -e CQHTTP_ACCESS_TOKEN= ac0f790e1fb74ebcaf45da77a6f9de47  `#Access Token` \
     -e CQHTTP_POST_MESSAGE_FORMAT= array  `# 回传消息时使用数组 (必选)` \
     richardchien/cqhttp: latest
```

然后使用 `ip:9000` 访问 `noVNC` 登录 ` 酷 Q` 即可.

新建 `QQ` 模块配置文件:

```
mkdir ~/.ehforwarderbot/profiles/default/milkice.qq
nano ~/.ehforwarderbot/profiles/default/milkice.qq/config.yaml
```

填入的代码大致如下:

```
Client: CoolQ                         #指定要使用的 QQ 客户端 (此处为 CoolQ)
CoolQ:
    type: HTTP                        #指定 efb-qq-slave 与酷 Q 通信的方式 现阶段仅支持 HTTP
    access_token: ac0f790e1fb74ebcaf45da77a6f9de47
    api_root: http://127.0.0.1:5700/  # 酷 Q API 接口地址 / 端口
    host: 127.0.0.1                   # efb-qq-slave 所监听的地址用于接收消息
    port: 8000                        # 同上
    is_pro: false                      # 若为酷 Q Pro 则为 true, 反之为 false
    air_option:                       # 包含于 air_option 的配置选项仅当 is_pro 为 false 时才有效
        upload_to_smms: true          # 将来自 EFB 主端 (通常是 Telegram) 的图片上传到 sm.ms 服务器并以链接的形式发送到 QQ 端
```

最后使用 `ehforwarderbot` 命令启动即可.

## Docker 安装

这里选择 `2` 个最新的 `Docker` 镜像, 也是官方推荐的, 项目地址:

```
#Telegram 收发 QQ 消息
https://github.com/Earth-Online/efb-qq-coolq-docker
#Telegram 收发微信消息
https://www.github.com/Mikubill/efb-wechat-docker
```

**1, 安装 Docker**

```
#CentOS 6
rpm -iUvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
yum update -y
yum -y install docker-io
service docker start
chkconfig docker on

#CentOS 7, Debian, Ubuntu
curl -sSL https://get.docker.com/ | sh
systemctl start docker
systemctl enable docker
```

**2, Telegram 收发 QQ 消息**
安装 `docker-compose`:

```
curl -L https://github.com/docker/compose/releases/download/1.25.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod + x /usr/local/bin/docker-compose
#验证是否安装成功
docker-compose --version
#返回以下信息即安装成功
docker-compose version 1.25.0, build 1110ad01
```

拉取 `Docker` 源码:

```
git clone https://github.com/Earth-Online/efb-qq-coolq-docker.git
cd efb-qq-coolq-docker
#编辑 config.yaml 配置文件
nano ehforward_config/profiles/default/blueset.telegram/config.yaml
```

修改如下:

```
#token 和 userid 参数获取方法查看上面的手动安装教程
token: " 你的 TG 机器人 Token"
admins:
- 你的 Userid
```

然后再编辑 `docker-compose.yml` 文件:

```
nano docker-compose.yml
```

修改如下:

```
- VNC_PASSWD= 你的密码
- COOLQ_ACCOUNT= 你的 qq 账号
```

后台启动:

```
#第一次启动会构建镜像, 所以会慢点
docker-compose up -d
```

然后打开 `ip:9801` 登陆 `novnc` 后完成 `coolq` 登陆操作. 如果该地址打不开, 请检查下防火墙.

**3, Telegram 收发微信消息**

```
#拉取源码
git clone https://github.com/mikubill/efb-wechat-docker.git
#构建镜像
cd efb-wechat-docker && docker build -t mikubill/efbwechat .
#启动镜像, TOKEN 为 TG 机器人 Token, ADMIN 为你的 Userid, 获取方法查看上面的手动安装教程
docker run -d -t --name "efbwechat" -e TOKEN= xxxx -e ADMIN= xxxx mikubill/efbwechat
```

最后获取微信登录验证码, 使用命令:

```
docker logs -f efbwechat
```

扫描登录即可.

最后这里都没有给微信添加额外的配置文件, 直接使用默认的微信配置, 如果想扩展微信功能的可以参考→[传送门](https://github.com/blueset/efb-wechat-slave#%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E4%BE%8B), 不过由于模块使用的微信网页版, 所以支持的功能是有限的, 比如: 没有朋友圈, 不能发语音, 位置等等, 一般来说也够用了, 至于 `QQ` 的话, 功能肯定受 ` 酷 Q` 限制, 暂时不能处理好友请求处理, 加群请求处理, 语音发送 / 接收等, 对于 `Facebook Messenger` 模块的话, 有需求的可以自己试试安装配置.

* * *

> 版权声明: 本文为原创文章, 版权归 [Rat's Blog](https://www.moerats.com/) 所有, 转载请注明出处!
>
> 本文链接: [https://www.moerats.com/archives/931/](https://www.moerats.com/archives/931/)
>
> 如教程需要更新, 或者相关链接出现 404, 可以在文章下面评论留言.

`Vultr` 新用户注册送 `100` 美元 /`16` 个机房按小时计费, 支持支付宝, [[点击查看](https://www.moerats.com/archives/543/)].

最后修改:2020 年 02 月 23 日 10 : 26 AM

© 著作权归作者所有

*   [下一篇](https://www.moerats.com/archives/933/ "Merger: 一个美观的微信 / 支付宝 /PayPal 等付款二维码合并程序 ")
*   [上一篇](https://www.moerats.com/archives/935/ " 一款跨平台的快速, 简单, 干净的视频下载器: Annie, 支持 Bilibili/Youtube 等多个网站 ")

#### 发表评论 [取消回复](https://www.moerats.com/archives/931/#respond-post-931)

评论 \*

私密评论



名称 \*

! [](https://cdn.v2ex.com/gravatar/d41d8cd98f00b204e9800998ecf8427e? s=65&r= G&d=)

邮箱 \*

地址

发表评论 提交中...

#### 122 条评论

1.   ! [](https://q.qlogo.cn/g? b= qq&nk=903323244&s=100)

    **pppppplh**

    June 16th, 2020 at 09:04 pm

    您好 我已经可以在 novnc 查到 QQ 的运行日志了, 但 telegram 上面还是没有任何的消息 . bot 的权限我也给了 请问你知道为什么吗

    [回复](https://www.moerats.com/archives/931/? replyTo=29481#respond-post-931)

    1.   ! [](https://q.qlogo.cn/g? b= qq&nk=145415544&s=100)

        **wshdik**

        July 26th, 2020 at 12:27 am

        **[@pppppplh](#comment-29481)**

        我用 docker 安装也有这个问题, 用普通方法 每隔一段时间就要重新启动一下很不方便

        [回复](https://www.moerats.com/archives/931/? replyTo=30131#respond-post-931)


2.   ! [](https://cdn.v2ex.com/gravatar/82a7f7a77b0ce992efd7177688a56c1e? s=65&r= G&d=)

    **LonANi**

    April 14th, 2020 at 08:20 pm

    求教: 最后一步执行 ehforwarderbot --profile qq 之后提示:
    2020-04-14 20:06:08,428 \[Level 99\]: ehforwarderbot.\_\_main\_\_ (\_\_main\_\_.init; \_\_main\_\_.py:85)

    Initializing slave milkice.qq...

    2020-04-14 20:06:08,855 \[Level 99\]: ehforwarderbot.\_\_main\_\_ (\_\_main\_\_.init; \_\_main\_\_.py:96)

    Slave channel QQ Slave (milkice.qq) # Default profile is initialized.

    2020-04-14 20:06:08,859 \[Level 99\]: ehforwarderbot.\_\_main\_\_ (\_\_main\_\_.init; \_\_main\_\_.py:99)

    Initializing master blueset.telegram...

    Traceback (most recent call last):
    File "/usr/local/bin/ehforwarderbot", line 11, in <module>

    sys.exit(main())

    File "/usr/local/lib/python3.6/dist-packages/ehforwarderbot/\_\_main\_\_.py", line 277, in main

    init(conf)

    File "/usr/local/lib/python3.6/dist-packages/ehforwarderbot/\_\_main\_\_.py", line 103, in init

    coordinator.add\_channel(module(instance\_id= instance\_id))

    File "/usr/local/lib/python3.6/dist-packages/efb\_telegram\_master/\_\_init\_\_.py", line 100, in init

    if 'WEBP' not in Image.ID or not WebPImagePlugin.SUPPORTED:

    AttributeError: module 'PIL.WebPImagePlugin' has no attribute 'SUPPORTED'

    不知道是什么情况, 希望博主能帮忙看一下, 先谢过大佬!

    [回复](https://www.moerats.com/archives/931/? replyTo=25653#respond-post-931)

3.   ! [](https://cdn.v2ex.com/gravatar/c03cdd819c5b6ab9572a967f82e0923d? s=65&r= G&d=)

    **[维安雨轩](https://ukenn.top)**

    March 10th, 2020 at 01:58 pm

    \[root@instance-7 efb-qq-coolq-docker\]# docker-compose up
    \-bash: docker-compose: command not found
    我 centos-7 用的谷歌云, 我尝试各种方法安装 docker-compose 结果还是这样...

    [回复](https://www.moerats.com/archives/931/? replyTo=24584#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/c03cdd819c5b6ab9572a967f82e0923d? s=65&r= G&d=)

        **[维安雨轩](https://ukenn.top)**

        March 10th, 2020 at 03:28 pm

        **[@维安雨轩](#comment-24584)**

        前面问题解决了, 又出现这个问题: ERROR: Volume bot-db declared as external, but could not be found. Please create the volume manually using docker volume create --name= bot-db and try again.

        [回复](https://www.moerats.com/archives/931/? replyTo=24585#respond-post-931)

        1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

            **[Rat's](http://www.moerats.com/)**博主

            March 10th, 2020 at 10:22 pm

            **[@维安雨轩](#comment-24585)**

            试试先用这个命令 docker volume create --name= bot-db

            [回复](https://www.moerats.com/archives/931/? replyTo=24595#respond-post-931)



4.   ! [](https://cdn.v2ex.com/gravatar/ef8a53083e397de49dd2ec57c8cd4172? s=65&r= G&d=)

    **carlning**

    February 23rd, 2020 at 09:10 am

    微信限制我登陆网页版, 我做到最后一步才发现这个问题, 建议写前面让大家先试试

    [回复](https://www.moerats.com/archives/931/? replyTo=23800#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        February 23rd, 2020 at 10:22 am

        **[@carlning](#comment-23800)**

        好的, 已添加

        [回复](https://www.moerats.com/archives/931/? replyTo=23801#respond-post-931)

        1.   ! [](https://cdn.v2ex.com/gravatar/ef8a53083e397de49dd2ec57c8cd4172? s=65&r= G&d=)

            **carlning**

            February 24th, 2020 at 09:22 pm

            **[@Rat's](#comment-23801)**

            微信限制有救吗?

            [回复](https://www.moerats.com/archives/931/? replyTo=23846#respond-post-931)

            1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

                **[Rat's](http://www.moerats.com/)**博主

                February 25th, 2020 at 12:00 am

                **[@carlning](#comment-23846)**

                申诉看看?

                [回复](https://www.moerats.com/archives/931/? replyTo=23850#respond-post-931)




5.   ! [](https://cdn.v2ex.com/gravatar/72c2de18eecbebe2857eea7708d3c308? s=65&r= G&d=)

    **lecx**

    December 4th, 2019 at 04:13 pm

    ehforwarderbot 后是这样的, 怎么解决欸! [](//www.moerats.com/usr/paopao/ 小红脸.png)
    Traceback (most recent call last):
    File "/usr/local/bin/ehforwarderbot", line 11, in <module>

    load\_entry\_point('ehforwarderbot==2.0.0b23.dev3', 'console\_scripts', 'ehforwarderbot') ()

    File "/usr/local/lib/python3.6/dist-packages/ehforwarderbot/\_\_main\_\_.py", line 263, in main

    conf = config.load\_config()

    File "/usr/local/lib/python3.6/dist-packages/ehforwarderbot/config.py", line 64, in load\_config

    channel = utils.locate\_module(i, 'slave')

    File "/usr/local/lib/python3.6/dist-packages/ehforwarderbot/utils.py", line 138, in locate\_module

    return i.load()

    File "/usr/local/lib/python3.6/dist-packages/pkg\_resources/\_\_init\_\_.py", line 2442, in load

    self.require(\*args, \*\*kwargs)

    File "/usr/local/lib/python3.6/dist-packages/pkg\_resources/\_\_init\_\_.py", line 2465, in require

    items = working\_set.resolve(reqs, env, installer, extras= self.extras)

    File "/usr/local/lib/python3.6/dist-packages/pkg\_resources/\_\_init\_\_.py", line 791, in resolve

    raise VersionConflict(dist, req).with\_context(dependent\_req)

    pkg\_resources.VersionConflict: (requests 2.18.4 (/usr/lib/python3/dist-packages), Requirement.parse('requests>=2.22
    .0'))

    [回复](https://www.moerats.com/archives/931/? replyTo=22300#respond-post-931)

    1.   ! [](https://q.qlogo.cn/g? b= qq&nk=1355239678&s=100)

        **Jessie**

        January 5th, 2020 at 01:11 pm

        **[@lecx](#comment-22300)**

        sudo pip3 install --upgrade docker-compose
        我用了这个就可以了

        [回复](https://www.moerats.com/archives/931/? replyTo=22867#respond-post-931)

    2.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        December 5th, 2019 at 11:20 am

        **[@lecx](#comment-22300)**

        看不出来, 你可以去这里搜下相关问题和解决方法: https://github.com/blueset/ehForwarderBot/issues

        [回复](https://www.moerats.com/archives/931/? replyTo=22317#respond-post-931)


6.   ! [](https://cdn.v2ex.com/gravatar/c449d0b4c28578217b7332eedceb5899? s=65&r= G&d=)

    **xxx**

    November 30th, 2019 at 02:59 pm

    不知道为什么, 装了被挖矿了, 心态都炸了

    [回复](https://www.moerats.com/archives/931/? replyTo=22179#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        December 1st, 2019 at 12:29 am

        **[@xxx](#comment-22179)**

        这个不会吧, 怎么看出被挖矿的

        [回复](https://www.moerats.com/archives/931/? replyTo=22192#respond-post-931)

        1.   ! [](https://cdn.v2ex.com/gravatar/c449d0b4c28578217b7332eedceb5899? s=65&r= G&d=)

            **xxx**

            December 2nd, 2019 at 09:38 pm

            **[@Rat's](#comment-22192)**

            那天加上你的博客, 我弄了三个人的, 不知道是哪个的有, 你的技术贴很多, 应该不是你的

            [回复](https://www.moerats.com/archives/931/? replyTo=22247#respond-post-931)

        2.   ! [](https://cdn.v2ex.com/gravatar/c449d0b4c28578217b7332eedceb5899? s=65&r= G&d=)

            **xxx**

            December 2nd, 2019 at 09:36 pm

            **[@Rat's](#comment-22192)**

            CPU 超载了 1800, GCP 都给我封了

            [回复](https://www.moerats.com/archives/931/? replyTo=22246#respond-post-931)

            1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

                **[Rat's](http://www.moerats.com/)**博主

                December 3rd, 2019 at 11:33 am

                **[@xxx](#comment-22246)**

                那不清楚了, 好像你是第一个说被挖矿的

                [回复](https://www.moerats.com/archives/931/? replyTo=22258#respond-post-931)




7.   ! [](https://cdn.v2ex.com/gravatar/01163c61279efbf08a27831eb5554057? s=65&r= G&d=)

    **小刀**

    November 27th, 2019 at 04:52 pm

    您好 博主 在最后一步输入 ehforwarderbot 时 提示 ValueError: Master Channel is not specified in the profile config 我 centos 7 系统, 这个怎么操作

    [回复](https://www.moerats.com/archives/931/? replyTo=22073#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        November 28th, 2019 at 12:30 am

        **[@小刀](#comment-22073)**

        好像说你的配置文件不对

        [回复](https://www.moerats.com/archives/931/? replyTo=22080#respond-post-931)


8.   ! [](https://cdn.v2ex.com/gravatar/ea97986c7bd38a586e9d949f8283b9e7? s=65&r= G&d=)

    **陈 q**

    November 23rd, 2019 at 11:39 pm

    扫描后出现这个样子, 是什么原因

    If the QR code was not shown correctly, please visit:
    https://login.weixin.qq.com/qrcode/wZNHt4Sh6A==
    Traceback (most recent call last):
    File "/usr/local/bin/ehforwarderbot", line 11, in <module>

    sys.exit(main())

    File "/usr/local/lib/python3.6/dist-packages/ehforwarderbot/\_\_main\_\_.py", line 271, in main

    init(conf)

    File "/usr/local/lib/python3.6/dist-packages/ehforwarderbot/\_\_main\_\_.py", line 91, in init

    coordinator.add\_channel(cls(instance\_id= instance\_id))

    File "/usr/local/lib/python3.6/dist-packages/efb\_wechat\_slave/\_\_init\_\_.py", line 152, in init

    self.authenticate('console\_qr\_code')

    File "/usr/local/lib/python3.6/dist-packages/efb\_wechat\_slave/\_\_init\_\_.py", line 544, in authenticate

    logout\_callback= self.exit\_callback)

    File "/usr/local/lib/python3.6/dist-packages/efb\_wechat\_slave/vendor/wxpy/api/bot.py", line 97, in init

    enhance\_webwx\_request(self)

    File "/usr/local/lib/python3.6/dist-packages/efb\_wechat\_slave/vendor/wxpy/utils/misc.py", line 327, in enhance\_webwx\_request

    '&pass\_ticket= {li\[pass\_ticket\]}'.format(li= login\_info)

    KeyError: 'wxsid'

    [回复](https://www.moerats.com/archives/931/? replyTo=22015#respond-post-931)

9.   ! [](https://cdn.v2ex.com/gravatar/ea97986c7bd38a586e9d949f8283b9e7? s=65&r= G&d=)

    **陈 q**

    November 23rd, 2019 at 10:35 pm

    请问这个阿里云主机可以吗, 一定要国外的 VPS 吗

    [回复](https://www.moerats.com/archives/931/? replyTo=22009#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        November 23rd, 2019 at 11:31 pm

        **[@陈 q](#comment-22009)**

        对, 需要访问 tg

        [回复](https://www.moerats.com/archives/931/? replyTo=22010#respond-post-931)


10.   ! [](https://q.qlogo.cn/g? b= qq&nk=740943170&s=100)

    **tang**

    November 19th, 2019 at 03:24 pm

    博主, 请问 ubuntu 下的微信代收和 docker 下的 qq 代收, 需要两个不同的 bot 么, 我出了一点问题, 在同一个 vps 上.

    [回复](https://www.moerats.com/archives/931/? replyTo=21921#respond-post-931)

11.   ! [](https://cdn.v2ex.com/gravatar/9d6bfa9d649e7743d2a64942419d591e? s=65&r= G&d=)

    **xushbo**

    November 19th, 2019 at 03:08 pm

    扫完二维码出现了
    Traceback (most recent call last):
    File "/usr/local/bin/ehforwarderbot", line 11, in <module>

    load\_entry\_point('ehforwarderbot==2.0.0b21', 'console\_scripts', 'ehforwarderbot') ()

    File "/usr/local/lib/python3.6/site-packages/ehforwarderbot/\_\_main\_\_.py", line 271, in main

    init(conf)

    File "/usr/local/lib/python3.6/site-packages/ehforwarderbot/\_\_main\_\_.py", line 91, in init

    coordinator.add\_channel(cls(instance\_id= instance\_id))

    File "/usr/local/lib/python3.6/site-packages/efb\_wechat\_slave/\_\_init\_\_.py", line 152, in init

    self.authenticate('console\_qr\_code')

    File "/usr/local/lib/python3.6/site-packages/efb\_wechat\_slave/\_\_init\_\_.py", line 544, in authenticate

    logout\_callback= self.exit\_callback)

    File "/usr/local/lib/python3.6/site-packages/efb\_wechat\_slave/vendor/wxpy/api/bot.py", line 97, in init

    enhance\_webwx\_request(self)

    File "/usr/local/lib/python3.6/site-packages/efb\_wechat\_slave/vendor/wxpy/utils/misc.py", line 327, in enhance\_webwx\_request

    '&pass\_ticket= {li\[pass\_ticket\]}'.format(li= login\_info)

    KeyError: 'wxsid'

    [回复](https://www.moerats.com/archives/931/? replyTo=21919#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/ea97986c7bd38a586e9d949f8283b9e7? s=65&r= G&d=)

        **陈 q**

        November 23rd, 2019 at 11:40 pm

        **[@xushbo](#comment-21919)**

        你这个问题解决了, 我也是出现同样问题

        [回复](https://www.moerats.com/archives/931/? replyTo=22016#respond-post-931)

    2.   ! [](https://cdn.v2ex.com/gravatar/9d6bfa9d649e7743d2a64942419d591e? s=65&r= G&d=)

        **xushbo**

        November 19th, 2019 at 03:11 pm

        **[@xushbo](#comment-21919)**

        怎么整都整不好, 一直出现 <error> <ret>1203</ret> <message> 为了你的帐号安全, 此微信号不能登录网页微信. 你可以使用 Windows 微信或 Mac 微信在电脑端登录.Windows 微信下载地址: https://pc.weixin.qq.com Mac 微信下载地址: https://mac.weixin.qq.com</message> </error>
        2019-11-15 06:46:29,584 \[Level 99\]: plugins.blueset.wechat.WeChatChannel (\_\_init\_\_.console\_qr\_code; \_\_init\_\_.py:207)

        EWS: Please scan the QR code with your camera, screenshots will not work. (oedZSPgaFg==, 400)

        [回复](https://www.moerats.com/archives/931/? replyTo=21920#respond-post-931)

        1.   ! [](https://cdn.v2ex.com/gravatar/3961bb34a42d37435d17d4ff4fa52339? s=65&r= G&d=)

            **cc**

            October 26th, 2020 at 11:31 am

            **[@xushbo](#comment-21920)**

            这种本来就不能登, 微信的限制.

            [回复](https://www.moerats.com/archives/931/? replyTo=31365#respond-post-931)



12.   ! [](https://q.qlogo.cn/g? b= qq&nk=740943170&s=100)

    **tang**

    November 19th, 2019 at 08:23 am

    大佬, 第二次登录的时候, 微信扫码成功之后, bot 说 login again when you are ready, 相当于扫完码后没登进去,

    Traceback (most recent call last):
    File "/usr/lib/python3.6/threading.py", line 916, in \_bootstrap\_inner

    self.run()

    File "/usr/lib/python3.6/threading.py", line 864, in run

    self.\_target(\*self.\_args, \*\*self.\_kwargs)

    File "/home/wiki/.local/lib/python3.6/site-packages/efb\_wechat\_slave/vendor/itchat/components/login.py", line 295, in maintain\_loop

    exitCallback()

    File "/home/wiki/.local/lib/python3.6/site-packages/efb\_wechat\_slave/\_\_init\_\_.py", line 244, in exit\_callback

    raise Exception(self.\_("Web WeChat logged your account out before master channel is ready."))

    Exception: Web WeChat logged your account out before master channel is ready.

    [回复](https://www.moerats.com/archives/931/? replyTo=21916#respond-post-931)

    1.   ! [](https://q.qlogo.cn/g? b= qq&nk=740943170&s=100)

        **tang**

        November 19th, 2019 at 08:43 am

        **[@tang](#comment-21916)**

        ok 已在此 issue 中 找到问题和方法, https://github.com/blueset/efb-wechat-slave/issues/61

        rm -r ~/.ehforwarderbot/profiles/default/blueset.w\*

        [回复](https://www.moerats.com/archives/931/? replyTo=21917#respond-post-931)


13.   ! [](https://cdn.v2ex.com/gravatar/9d6bfa9d649e7743d2a64942419d591e? s=65&r= G&d=)

    **xushbo**

    November 15th, 2019 at 07:51 pm

    <error> <ret>1203</ret> <message> 为了你的帐号安全, 此微信号不能登录网页微信. 你可以使用 Windows 微信或 Mac 微信在电脑端登录.Windows 微信下载地址: https://pc.weixin.qq.com Mac 微信下载地址: https://mac.weixin.qq.com</message> </error>
    2019-11-15 06:46:29,584 \[Level 99\]: plugins.blueset.wechat.WeChatChannel (\_\_init\_\_.console\_qr\_code; \_\_init\_\_.py:207)

    EWS: Please scan the QR code with your camera, screenshots will not work. (oedZSPgaFg==, 400)

    好不容易弄好了 出现这个情况

    [回复](https://www.moerats.com/archives/931/? replyTo=21853#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        November 15th, 2019 at 10:34 pm

        **[@xushbo](#comment-21853)**

        这个要你直接扫描二维码, 不要扫描截图的翻译

        [回复](https://www.moerats.com/archives/931/? replyTo=21856#respond-post-931)


14.   ! [](https://cdn.v2ex.com/gravatar/9d6bfa9d649e7743d2a64942419d591e? s=65&r= G&d=)

    **xushbo**

    November 14th, 2019 at 07:41 pm

    大佬最后一步了, 出现这个问题
    Traceback (most recent call last):
    File "/usr/local/bin/ehforwarderbot", line 11, in <module>

    sys.exit(main())

    File "/usr/local/lib/python3.6/site-packages/ehforwarderbot/\_\_main\_\_.py", line 262, in main

    conf = config.load\_config()

    File "/usr/local/lib/python3.6/site-packages/ehforwarderbot/config.py", line 47, in load\_config

    channel = utils.locate\_module(data\['master\_channel'\], 'master')

    File "/usr/local/lib/python3.6/site-packages/ehforwarderbot/utils.py", line 138, in locate\_module

    return i.load()

    File "/usr/local/lib/python3.6/site-packages/pkg\_resources/\_\_init\_\_.py", line 2442, in load

    self.require(\*args, \*\*kwargs)

    File "/usr/local/lib/python3.6/site-packages/pkg\_resources/\_\_init\_\_.py", line 2465, in require

    items = working\_set.resolve(reqs, env, installer, extras= self.extras)

    File "/usr/local/lib/python3.6/site-packages/pkg\_resources/\_\_init\_\_.py", line 786, in resolve

    raise DistributionNotFound(req, requirers)

    pkg\_resources.DistributionNotFound: The 'cairocffi' distribution was not found and is required by cairosvg

    [回复](https://www.moerats.com/archives/931/? replyTo=21835#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        November 14th, 2019 at 11:43 pm

        **[@xushbo](#comment-21835)**

        看报错好像是 cairocffi 没安装, 试试这个命令看看 pip3 install cairocffi

        [回复](https://www.moerats.com/archives/931/? replyTo=21839#respond-post-931)


15.   ! [](https://cdn.v2ex.com/gravatar/333aa7970f892d8c6a75c55e3c6efe70? s=65&r= G&d=)

    **你好, 跪求帮忙啊**

    October 12th, 2019 at 03:20 am

    conf = config.load\_config()

    File "/usr/local/lib/python3.6/site-packages/ehforwarderbot/config.py", line 47, in load\_config

    channel = utils.locate\_module(data\['master\_channel'\], 'master')

    File "/usr/local/lib/python3.6/site-packages/ehforwarderbot/utils.py", line 138, in locate\_module

    return i.load()

    File "/usr/local/lib/python3.6/site-packages/pkg\_resources/\_\_init\_\_.py", line 2346, in load

    return self.resolve()

    File "/usr/local/lib/python3.6/site-packages/pkg\_resources/\_\_init\_\_.py", line 2352, in resolve

    module = \_\_import\_\_(self.module\_name, fromlist=\['\_\_name\_\_'\], level=0)

    File "/usr/local/lib/python3.6/site-packages/efb\_telegram\_master/\_\_init\_\_.py", line 28, in <module>

    from . import utils as etm\_utils

    File "/usr/local/lib/python3.6/site-packages/efb\_telegram\_master/utils.py", line 10, in <module>

    from tgs.parsers.tgs import parse\_tgs

    File "/usr/local/lib/python3.6/site-packages/tgs/\_\_init\_\_.py", line 1, in <module>

    from . import objects, parsers, utils, exporters, nvector

    File "/usr/local/lib/python3.6/site-packages/tgs/parsers/\_\_init\_\_.py", line 1, in <module>

    from . import svg, tgs, sif

    File "/usr/local/lib/python3.6/site-packages/tgs/parsers/svg/\_\_init\_\_.py", line 1, in <module>

    from .importer import parse\_svg\_etree, parse\_svg\_file

    File "/usr/local/lib/python3.6/site-packages/tgs/parsers/svg/importer.py", line 8, in <module>

    from ...utils.ellipse import Ellipse

    File "/usr/local/lib/python3.6/site-packages/tgs/utils/\_\_init\_\_.py", line 1, in <module>

    from . import animation, ellipse, ik, linediff, restructure, script, stripper

    File "/usr/local/lib/python3.6/site-packages/tgs/utils/linediff.py", line 3, in <module>

    from ..exporters import prettyprint

    File "/usr/local/lib/python3.6/site-packages/tgs/exporters/\_\_init\_\_.py", line 16, in <module>

    from . import cairo, gif

    File "/usr/local/lib/python3.6/site-packages/tgs/exporters/cairo.py", line 1, in <module>

    import cairosvg

    File "/usr/local/lib/python3.6/site-packages/cairosvg/\_\_init\_\_.py", line 42, in <module>

    from . import surface # noqa isort: skip

    File "/usr/local/lib/python3.6/site-packages/cairosvg/surface.py", line 25, in <module>

    import cairocffi as cairo

    File "/usr/local/lib/python3.6/site-packages/cairocffi/\_\_init\_\_.py", line 50, in <module>

    ('libcairo.so', 'libcairo.2.dylib', 'libcairo-2.dll'))

    File "/usr/local/lib/python3.6/site-packages/cairocffi/\_\_init\_\_.py", line 45, in dlopen

    raise OSError(error\_message) # pragma: no cover

    OSError: no library called "cairo" was found
    no library called "libcairo-2" was found
    cannot load library 'libcairo.so': Error loading shared library libcairo.so: No such file or directory
    cannot load library 'libcairo.2.dylib': Error loading shared library libcairo.2.dylib: No such file or directory
    cannot load library 'libcairo-2.dll': Error loading shared library libcairo-2.dll: No such file or directory

    [回复](https://www.moerats.com/archives/931/? replyTo=21162#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/333aa7970f892d8c6a75c55e3c6efe70? s=65&r= G&d=)

        **你好, 跪求帮忙啊**

        October 12th, 2019 at 03:22 am

        **[@你好, 跪求帮忙啊](#comment-21162)**

        请问这咋办啊?

        [回复](https://www.moerats.com/archives/931/? replyTo=21163#respond-post-931)

        1.   ! [](https://cdn.v2ex.com/gravatar/333aa7970f892d8c6a75c55e3c6efe70? s=65&r= G&d=)

            **你好, 跪求帮忙啊**

            October 12th, 2019 at 01:44 pm

            **[@你好, 跪求帮忙啊](#comment-21163)**

            现在出现扫码界面了 可为什么 扫码登录后 不转发消息呢?

            [回复](https://www.moerats.com/archives/931/? replyTo=21172#respond-post-931)

            1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

                **[Rat's](http://www.moerats.com/)**博主

                October 12th, 2019 at 11:57 pm

                **[@你好, 跪求帮忙啊](#comment-21172)**

                有日志信息吗?

                [回复](https://www.moerats.com/archives/931/? replyTo=21194#respond-post-931)

                1.   ! [](https://cdn.v2ex.com/gravatar/333aa7970f892d8c6a75c55e3c6efe70? s=65&r= G&d=)

                    **你好, 跪求帮忙啊**

                    October 14th, 2019 at 10:11 pm

                    **[@Rat's](#comment-21194)**

                    If you cannot read the QR code above, please visit the following URL:
                    https://login.weixin.qq.com/qrcode/Qbz7VT31zQ==
                    14-10-2019:14:06:23,434 CRITICAL \[main.py:86\]

                    Slave channel WeChat Slave (eh\_wechat\_slave) initialized.

                    14-10-2019:14:06:23,435 CRITICAL \[main.py:87\]

                    Initializing master ('plugins.eh\_telegram\_master', 'TelegramChannel')...

                    Imageio: 'ffmpeg-linux64-v3.3.1' was not found on your computer; downloading it now.
                    Try 1. Download from https://github.com/imageio/imageio-binaries/raw/master/ffmpeg/ffmpeg-linux64-v3.3.1 (43.8 MB)
                    Downloading: 45929032/45929032 bytes (100.0%)
                    Done
                    14-10-2019:14:06:32,452 CRITICAL \[main.py:90\]

                    Master channel Telegram Master (eh\_telegram\_master) initialized.

                    14-10-2019:14:06:32,452 CRITICAL \[main.py:92\]

                    All channels initialized.

                    File saved as /root/.imageio/ffmpeg/ffmpeg-linux64-v3.3.1.

                    [回复](https://www.moerats.com/archives/931/? replyTo=21237#respond-post-931)

                    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

                        **[Rat's](http://www.moerats.com/)**博主

                        October 15th, 2019 at 09:02 am

                        **[@你好, 跪求帮忙啊](#comment-21237)**

                        这个提示是说你没安装 ffmpeg.

                        [回复](https://www.moerats.com/archives/931/? replyTo=21238#respond-post-931)

                        1.   ! [](https://cdn.v2ex.com/gravatar/333aa7970f892d8c6a75c55e3c6efe70? s=65&r= G&d=)

                            **你好, 跪求帮忙啊**

                            October 17th, 2019 at 12:26 am

                            **[@Rat's](#comment-21238)**

                            楼主 现在你的 dock 安装教程 还有用吗? 我用的别的教程安装的 dock 最后出现二维码了 可机器人就是不转发消息

                            [回复](https://www.moerats.com/archives/931/? replyTo=21267#respond-post-931)

                            1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

                                **[Rat's](http://www.moerats.com/)**博主

                                October 17th, 2019 at 11:40 pm

                                **[@你好, 跪求帮忙啊](#comment-21267)**

                                一般能打开就么得问题, 有空我再实践一下.

                                [回复](https://www.moerats.com/archives/931/? replyTo=21293#respond-post-931)








16.   ! [](https://cdn.v2ex.com/gravatar/ba19b771f2738b6637d361b8140baf5e? s=65&r= G&d=)

    **[tabris](http://tabris.top)**

    October 6th, 2019 at 09:14 pm

    VPS 不再墙外 没有关系的吧?

    [回复](https://www.moerats.com/archives/931/? replyTo=21055#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        October 12th, 2019 at 11:56 pm

        **[@tabris](#comment-21055)**

        这个需要墙外的 vps

        [回复](https://www.moerats.com/archives/931/? replyTo=21192#respond-post-931)


17.   ! [](https://cdn.v2ex.com/gravatar/edab35c96f51a59ad151b0fb0fa3abdf? s=65&r= G&d=)

    **Adoc**

    September 20th, 2019 at 02:00 pm

    您好, 博主, 可以绑定多个微信号, 绑定多个机器人吗

    [回复](https://www.moerats.com/archives/931/? replyTo=20662#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        October 12th, 2019 at 11:56 pm

        **[@Adoc](#comment-20662)**

        可以绑定多个微信

        [回复](https://www.moerats.com/archives/931/? replyTo=21193#respond-post-931)


18.   ! [](https://cdn.v2ex.com/gravatar/60cb6e1b01b60b6ebced1ba02b1bb7bb? s=65&r= G&d=)

    **stardif**

    August 3rd, 2019 at 10:49 pm

    Traceback (most recent call last):
    File "/usr/local/bin/ehforwarderbot", line 10, in <module>

    sys.exit(main())

    File "/usr/local/lib/python3.7/site-packages/ehforwarderbot/\_\_main\_\_.py", line 267, in main

    init(conf)

    File "/usr/local/lib/python3.7/site-packages/ehforwarderbot/\_\_main\_\_.py", line 91, in init

    coordinator.add\_channel(cls(instance\_id= instance\_id))

    File "/usr/local/lib/python3.7/site-packages/efb\_wechat\_slave/\_\_init\_\_.py", line 174, in init

    self.authenticate('console\_qr\_code')

    File "/usr/local/lib/python3.7/site-packages/efb\_wechat\_slave/\_\_init\_\_.py", line 553, in authenticate

    logout\_callback= self.exit\_callback)

    File "/usr/local/lib/python3.7/site-packages/efb\_wechat\_slave/wxpy/api/bot.py", line 97, in init

    enhance\_webwx\_request(self)

    File "/usr/local/lib/python3.7/site-packages/efb\_wechat\_slave/wxpy/utils/misc.py", line 327, in enhance\_webwx\_request

    '&pass\_ticket= {li\[pass\_ticket\]}'.format(li= login\_info)

    KeyError: 'wxsid'
    2019-08-03 22:19:16,671 \[Level 99\]: ehforwarderbot.\_\_main\_\_ (\_\_main\_\_.init; \_\_main\_\_.py:85)

    Initializing slave blueset.wechat...

    Traceback (most recent call last):
    File "/usr/local/bin/ehforwarderbot", line 10, in <module>

    sys.exit(main())

    File "/usr/local/lib/python3.7/site-packages/ehforwarderbot/\_\_main\_\_.py", line 267, in main

    init(conf)

    File "/usr/local/lib/python3.7/site-packages/ehforwarderbot/\_\_main\_\_.py", line 91, in init

    coordinator.add\_channel(cls(instance\_id= instance\_id))

    File "/usr/local/lib/python3.7/site-packages/efb\_wechat\_slave/\_\_init\_\_.py", line 174, in init

    self.authenticate('console\_qr\_code')

    File "/usr/local/lib/python3.7/site-packages/efb\_wechat\_slave/\_\_init\_\_.py", line 553, in authenticate

    logout\_callback= self.exit\_callback)

    File "/usr/local/lib/python3.7/site-packages/efb\_wechat\_slave/wxpy/api/bot.py", line 87, in init

    loginCallback= login\_callback, exitCallback= logout\_callback

    File "/usr/local/lib/python3.7/site-packages/itchat/components/register.py", line 29, in auto\_login

    loginCallback= loginCallback, exitCallback= exitCallback):

    File "/usr/local/lib/python3.7/site-packages/itchat/components/hotreload.py", line 54, in load\_login\_status

    self.loginInfo\['User'\] = templates.User(self.loginInfo\['User'\])

    KeyError: 'User'

    请问这是哪里有问题?

    [回复](https://www.moerats.com/archives/931/? replyTo=17317#respond-post-931)

    1.   ! [](https://q.qlogo.cn/g? b= qq&nk=1301449186&s=100)

        **Lison**

        September 29th, 2019 at 01:34 am

        **[@stardif](#comment-17317)**

        Traceback (most recent call last):
        File "/usr/local/bin/ehforwarderbot", line 11, in <module>

        sys.exit(main())

        File "/usr/local/lib/python3.6/site-packages/ehforwarderbot/\_\_main\_\_.py", line 262, in main

        conf = config.load\_config()

        File "/usr/local/lib/python3.6/site-packages/ehforwarderbot/config.py", line 47, in load\_config

        channel = utils.locate\_module(data\['master\_channel'\], 'master')

        File "/usr/local/lib/python3.6/site-packages/ehforwarderbot/utils.py", line 138, in locate\_module

        return i.load()

        File "/usr/local/lib/python3.6/site-packages/pkg\_resources/\_\_init\_\_.py", line 2443, in load

        return self.resolve()

        File "/usr/local/lib/python3.6/site-packages/pkg\_resources/\_\_init\_\_.py", line 2449, in resolve

        module = \_\_import\_\_(self.module\_name, fromlist=\['\_\_name\_\_'\], level=0)

        File "/usr/local/lib/python3.6/site-packages/efb\_telegram\_master/\_\_init\_\_.py", line 28, in <module>

        from . import utils as etm\_utils

        File "/usr/local/lib/python3.6/site-packages/efb\_telegram\_master/utils.py", line 10, in <module>

        from tgs.parsers.tgs import parse\_tgs

        File "/usr/local/lib/python3.6/site-packages/tgs/\_\_init\_\_.py", line 1, in <module>

        from . import objects, parsers, utils, exporters, nvector

        File "/usr/local/lib/python3.6/site-packages/tgs/parsers/\_\_init\_\_.py", line 1, in <module>

        from . import svg, tgs, sif

        File "/usr/local/lib/python3.6/site-packages/tgs/parsers/svg/\_\_init\_\_.py", line 1, in <module>

        from .importer import parse\_svg\_etree, parse\_svg\_file

        File "/usr/local/lib/python3.6/site-packages/tgs/parsers/svg/importer.py", line 8, in <module>

        from ...utils.ellipse import Ellipse

        File "/usr/local/lib/python3.6/site-packages/tgs/utils/\_\_init\_\_.py", line 1, in <module>

        from . import animation, ellipse, ik, linediff, restructure, script, stripper

        File "/usr/local/lib/python3.6/site-packages/tgs/utils/linediff.py", line 3, in <module>

        from ..exporters import prettyprint

        File "/usr/local/lib/python3.6/site-packages/tgs/exporters/\_\_init\_\_.py", line 16, in <module>

        from . import cairo, gif

        File "/usr/local/lib/python3.6/site-packages/tgs/exporters/cairo.py", line 1, in <module>

        import cairosvg

        File "/usr/local/lib/python3.6/site-packages/cairosvg/\_\_init\_\_.py", line 42, in <module>

        from . import surface # noqa isort: skip

        File "/usr/local/lib/python3.6/site-packages/cairosvg/surface.py", line 25, in <module>

        import cairocffi as cairo

        File "/usr/local/lib/python3.6/site-packages/cairocffi/\_\_init\_\_.py", line 50, in <module>

        ('libcairo.so', 'libcairo.2.dylib', 'libcairo-2.dll'))

        File "/usr/local/lib/python3.6/site-packages/cairocffi/\_\_init\_\_.py", line 45, in dlopen

        raise OSError(error\_message) # pragma: no cover

        OSError: no library called "cairo" was found
        no library called "libcairo-2" was found
        cannot load library 'libcairo.so': libcairo.so: cannot open shared object file: No such file or directory
        cannot load library 'libcairo.2.dylib': libcairo.2.dylib: cannot open shared object file: No such file or directory
        cannot load library 'libcairo-2.dll': libcairo-2.dll: cannot open shared object file: No such file or directory
        我也是这个问题, 输入 ehforwarderbot 就会报错 , 我也把 pip3 的 bin 目录加入到系统 PATH 环境变量了,! [](//www.moerats.com/usr/paopao/ 泪.png) 球球大佬们的帮助

        [回复](https://www.moerats.com/archives/931/? replyTo=20887#respond-post-931)

        1.   ! [](https://cdn.v2ex.com/gravatar/d6c78d34a6657f2a6d1419e0ca16d4c8? s=65&r= G&d=)

            **jiz4oh**

            September 30th, 2019 at 11:47 am

            **[@Lison](#comment-20887)**

            使用以下命令安装依赖库
            yum install cairo-devel
            yum install libtiff\*

            [回复](https://www.moerats.com/archives/931/? replyTo=20981#respond-post-931)

        2.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

            **[Rat's](http://www.moerats.com/)**博主

            September 29th, 2019 at 10:41 pm

            **[@Lison](#comment-20887)**

            ! [](//www.moerats.com/usr/paopao/ 黑线.png) 好像加载的时候缺少共享库, 你可以换个系统操作看看.

            [回复](https://www.moerats.com/archives/931/? replyTo=20970#respond-post-931)

            1.   ! [](https://q.qlogo.cn/g? b= qq&nk=1301449186&s=100)

                **Lison**

                September 29th, 2019 at 11:57 pm

                **[@Rat's](#comment-20970)**

                我找到原因了, 有些 vps 没有安装 Cairo 图形库, 搭建之后就能显示二维码了! [](//www.moerats.com/usr/paopao/ 吐舌.png)

                [回复](https://www.moerats.com/archives/931/? replyTo=20975#respond-post-931)

                1.   ! [](https://cdn.v2ex.com/gravatar/509e8cd2ea5e58119e42d19900f8acb1? s=65&r= G&d=)

                    **pq8o3q**

                    October 6th, 2019 at 10:52 pm

                    **[@Lison](#comment-20975)**

                    大佬, 请问下您是怎么搞好的, 可以说一下命令吗, 我谷歌搜了下, 安装了还是不行, 非常感谢

                    [回复](https://www.moerats.com/archives/931/? replyTo=21058#respond-post-931)

                    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

                        **[Rat's](http://www.moerats.com/)**博主

                        October 6th, 2019 at 10:56 pm

                        **[@pq8o3q](#comment-21058)**

                        这是上面提供的命令:
                        yum install cairo-devel
                        yum install libtiff\*

                        [回复](https://www.moerats.com/archives/931/? replyTo=21059#respond-post-931)

                        1.   ! [](https://cdn.v2ex.com/gravatar/509e8cd2ea5e58119e42d19900f8acb1? s=65&r= G&d=)

                            **pq8o3q**

                            October 6th, 2019 at 11:00 pm

                            **[@Rat's](#comment-21059)**

                            还有就是就算使用 docker, 最后也会出现这种错误, 用完那两个命令也是依旧会

                            [回复](https://www.moerats.com/archives/931/? replyTo=21064#respond-post-931)

                        2.   ! [](https://cdn.v2ex.com/gravatar/509e8cd2ea5e58119e42d19900f8acb1? s=65&r= G&d=)

                            **pq8o3q**

                            October 6th, 2019 at 10:59 pm

                            **[@Rat's](#comment-21059)**

                            博主, 我也试了, 但还是出现上面的错误, 不知到怎么回事.centos7 原版系统和 dd 萌咖大佬的 debian9, 都会出现错误, 很郁闷...

                            [回复](https://www.moerats.com/archives/931/? replyTo=21063#respond-post-931)



                2.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

                    **[Rat's](http://www.moerats.com/)**博主

                    September 30th, 2019 at 07:59 pm

                    **[@Lison](#comment-20975)**

                    ! [](//www.moerats.com/usr/alu/ 高兴.png) 解决好了就行

                    [回复](https://www.moerats.com/archives/931/? replyTo=20989#respond-post-931)





19.   ! [](https://cdn.v2ex.com/gravatar/21c1ae67a5947d427d50d13ebe94eaf0? s=65&r= G&d=)

    **哥哥**

    July 26th, 2019 at 08:06 pm

    如果配置了多个微信同时接收. 如果其中一个掉线了 (手机微信换了设备), 需要重新登录. 那么如何单独重新启动这一个微信了?

    [回复](https://www.moerats.com/archives/931/? replyTo=17088#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        July 26th, 2019 at 11:14 pm

        **[@哥哥](#comment-17088)**

        这个不太清楚, 没试过单独启动一个

        [回复](https://www.moerats.com/archives/931/? replyTo=17091#respond-post-931)

        1.   ! [](https://cdn.v2ex.com/gravatar/21c1ae67a5947d427d50d13ebe94eaf0? s=65&r= G&d=)

            **哥哥**

            July 27th, 2019 at 02:41 pm

            **[@Rat's](#comment-17091)**

            我经常遇到重启失败, 采取的措施就删掉~/.ehforwarderbot/profiles/default/blueset.wechat
            但是这样有一个很大的问题, 就是绑定好的链接会失效, 请问你有更好的方法吗?

            [回复](https://www.moerats.com/archives/931/? replyTo=17108#respond-post-931)

            1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

                **[Rat's](http://www.moerats.com/)**博主

                July 27th, 2019 at 08:47 pm

                **[@哥哥](#comment-17108)**

                其实么有, 我用的时间不长, 很多情景还没遇到.

                [回复](https://www.moerats.com/archives/931/? replyTo=17111#respond-post-931)

                1.   ! [](https://cdn.v2ex.com/gravatar/21c1ae67a5947d427d50d13ebe94eaf0? s=65&r= G&d=)

                    **哥哥**

                    July 28th, 2019 at 02:21 pm

                    **[@Rat's](#comment-17111)**

                    好的, 谢谢分享, 有新功能, 你可以继续更新的! 很期待

                    [回复](https://www.moerats.com/archives/931/? replyTo=17127#respond-post-931)





20.   ! [](https://q.qlogo.cn/g? b= qq&nk=1085727448&s=100)

    **Bruce**

    June 21st, 2019 at 01:13 pm

    请问博主, 在微信端已经屏蔽的群消息怎么设置不转发到 tg

    [回复](https://www.moerats.com/archives/931/? replyTo=16158#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        June 21st, 2019 at 08:08 pm

        **[@Bruce](#comment-16158)**

        这个就不太清楚了, 对使用方面研究不太深.

        [回复](https://www.moerats.com/archives/931/? replyTo=16159#respond-post-931)


21.   ! [](https://cdn.v2ex.com/gravatar/ee494a82971b9818acd3dc823e3ccb11? s=65&r= G&d=)

    **eatneko**

    May 17th, 2019 at 09:46 am

    博主, 请问一下, 这个运行需要多大内存.

    [回复](https://www.moerats.com/archives/931/? replyTo=15260#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        May 17th, 2019 at 09:19 pm

        **[@eatneko](#comment-15260)**

        我 512 内存, 添加了点虚拟内存就可以了

        [回复](https://www.moerats.com/archives/931/? replyTo=15270#respond-post-931)

        1.   ! [](https://cdn.v2ex.com/gravatar/ee494a82971b9818acd3dc823e3ccb11? s=65&r= G&d=)

            **eatneko**

            May 24th, 2019 at 04:28 pm

            **[@Rat's](#comment-15270)**

            博主, 喔 debain 系统的 Python 安装错了, 请问, 怎么卸载, 安装了 Ubuntu 系统的

            [回复](https://www.moerats.com/archives/931/? replyTo=15458#respond-post-931)

            1.   ! [](https://cdn.v2ex.com/gravatar/ee494a82971b9818acd3dc823e3ccb11? s=65&r= G&d=)

                **eatneko**

                May 24th, 2019 at 04:57 pm

                **[@eatneko](#comment-15458)**

                用这命令卸载 apt-get remove --auto-remove python3.\*, 重装 python3.6.4 后, 显示 pip3 不存在

                [回复](https://www.moerats.com/archives/931/? replyTo=15459#respond-post-931)




22.   ! [](https://cdn.v2ex.com/gravatar/e94b8284c69a2fadd8566e1d34316a8f? s=65&r= G&d=)

    **來不及**

    May 8th, 2019 at 01:08 am

    好久不见, 一切祝好,
    博主有没有 Linux 挂载硬盘方面的经验,
    可以分享一下吗?

    [回复](https://www.moerats.com/archives/931/? replyTo=15025#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        May 8th, 2019 at 06:15 pm

        **[@來不及](#comment-15025)**

        我以前发过挂载硬盘教程, 地址: https://www.moerats.com/archives/514/

        [回复](https://www.moerats.com/archives/931/? replyTo=15038#respond-post-931)


23.   ! [](https://cdn.v2ex.com/gravatar/4d8fd61c3c2795322ca45e33a71690b8? s=65&r= G&d=)

    **西瓜**

    May 5th, 2019 at 06:50 pm

    微信多开的时候出现如下错误, 感觉的 PYTHON 的问题. 请问大佬怎么解决
    Exception in thread Thread-2:
    Traceback (most recent call last):
    File "/usr/lib/python3.6/threading.py", line 916, in \_bootstrap\_inner
    self.run()
    File "/usr/lib/python3.6/threading.py", line 864, in run
    self.\_target(self.\_args, \*self.\_kwargs)
    File "/usr/local/lib/python3.6/dist-packages/itchat/components/login.py", line 281, in maintain\_loop
    exitCallback()
    File "/usr/local/lib/python3.6/dist-packages/efb\_wechat\_slave/init.py", line 264, in exit\_callback
    if not getattr(coordinator, 'master', default= None):
    TypeError: getattr() takes no keyword arguments

    [回复](https://www.moerats.com/archives/931/? replyTo=14951#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        May 5th, 2019 at 10:04 pm

        **[@西瓜](#comment-14951)**

        这个我也不是很清楚, 没遇到过, 可以去 Github 反馈下.

        [回复](https://www.moerats.com/archives/931/? replyTo=14953#respond-post-931)

        1.   ! [](https://cdn.v2ex.com/gravatar/4d8fd61c3c2795322ca45e33a71690b8? s=65&r= G&d=)

            **西瓜**

            May 5th, 2019 at 11:47 pm

            **[@Rat's](#comment-14953)**

            那边没人回我, 桑心

            [回复](https://www.moerats.com/archives/931/? replyTo=14966#respond-post-931)



24.   ! [](https://cdn.v2ex.com/gravatar/3f09a86a37bbf5332bd7d13222ffd984? s=65&r= G&d=)

    **asassss**

    May 5th, 2019 at 04:04 pm

    <script>

    while (true) { alert('Hello') }

    </script>

    [回复](https://www.moerats.com/archives/931/? replyTo=14945#respond-post-931)

25.   ! [](https://cdn.v2ex.com/gravatar/f1178ccee83f33ee4929e0edbe2ead68? s=65&r= G&d=)

    **Toast**

    April 30th, 2019 at 09:01 pm

    可以设置一个 BOT 加上一个 QQ 还有一个微信 这样的设置吗

    [回复](https://www.moerats.com/archives/931/? replyTo=14783#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        April 30th, 2019 at 11:45 pm

        **[@Toast](#comment-14783)**

        可以啊, 你试试

        [回复](https://www.moerats.com/archives/931/? replyTo=14794#respond-post-931)


26.   ! [](https://cdn.v2ex.com/gravatar/2cebb47135235c18f8d3917d7bc8b20b? s=65&r= G&d=)

    **su**

    April 26th, 2019 at 04:28 pm

    为什么一切都安装好了, 但是还是没有消息提示呢?

    [回复](https://www.moerats.com/archives/931/? replyTo=14601#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        April 27th, 2019 at 12:18 am

        **[@su](#comment-14601)**

        因为你有步骤出了问题

        [回复](https://www.moerats.com/archives/931/? replyTo=14622#respond-post-931)


27.   ! [](https://cdn.v2ex.com/gravatar/2cebb47135235c18f8d3917d7bc8b20b? s=65&r= G&d=)

    **su**

    April 26th, 2019 at 02:56 pm

    Traceback (most recent call last):
    File "/usr/local/bin/ehforwarderbot", line 11, in <module>

    load\_entry\_point('ehforwarderbot==2.0.0b16.dev1', 'console\_scripts', 'ehforwarderbot') ()

    File "/usr/local/lib/python3.6/site-packages/ehforwarderbot/\_\_main\_\_.py", line 260, in main

    conf = config.load\_config()

    File "/usr/local/lib/python3.6/site-packages/ehforwarderbot/config.py", line 37, in load\_config

    with open(conf\_path) as f:

    IsADirectoryError: \[Errno 21\] Is a directory: '/root/.ehforwarderbot/profiles/default/config.yaml'

    [回复](https://www.moerats.com/archives/931/? replyTo=14598#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        April 27th, 2019 at 12:18 am

        **[@su](#comment-14598)**

        看下 config.yaml 配置文件对不对

        [回复](https://www.moerats.com/archives/931/? replyTo=14621#respond-post-931)


28.   ! [](https://q.qlogo.cn/g? b= qq&nk=1594176339&s=100)

    **树先生**

    April 23rd, 2019 at 12:18 am

    大佬, 我主要过来歪个楼, 比较好的博客模版您有没有 (比如您网站这样的) (手动外头! [](//www.moerats.com/usr/paopao/ 太开心.png))

    [回复](https://www.moerats.com/archives/931/? replyTo=14464#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        April 23rd, 2019 at 01:43 pm

        **[@树先生](#comment-14464)**

        不知道啥叫好看, 你可以百度下.

        [回复](https://www.moerats.com/archives/931/? replyTo=14473#respond-post-931)


29.   ! [](https://cdn.v2ex.com/gravatar/d219af79b45e5891507fda4c4c2139a0? s=65&r= G&d=)

    **1**

    April 22nd, 2019 at 08:10 pm

    想问问手机版电报, 如何 " 换行即发送 "...

    每次都要点发送按钮, 好麻烦的!

    [回复](https://www.moerats.com/archives/931/? replyTo=14457#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        April 23rd, 2019 at 12:07 am

        **[@1](#comment-14457)**

        不是很清楚, 很少玩电报, 一般水教程的时候才打开下

        [回复](https://www.moerats.com/archives/931/? replyTo=14462#respond-post-931)


30.   ! [](https://q.qlogo.cn/g? b= qq&nk=1392575940&s=100)

    **[美国主机](https://www.yd631.com/)**

    April 22nd, 2019 at 07:01 pm

    朋友 交换链接吗

    [回复](https://www.moerats.com/archives/931/? replyTo=14455#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        April 23rd, 2019 at 12:08 am

        **[@美国主机](#comment-14455)**

        暂时不换了

        [回复](https://www.moerats.com/archives/931/? replyTo=14463#respond-post-931)


31.   ! [](https://q.qlogo.cn/g? b= qq&nk=1327200145&s=100)

    **chen**

    April 22nd, 2019 at 11:32 am

    博主, 我想问一下搭建服务端的服务器必须要能越过长城才可以正常接收消息吗?

    [回复](https://www.moerats.com/archives/931/? replyTo=14436#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        April 22nd, 2019 at 12:20 pm

        **[@chen](#comment-14436)**

        应该是这样的

        [回复](https://www.moerats.com/archives/931/? replyTo=14437#respond-post-931)

        1.   ! [](https://q.qlogo.cn/g? b= qq&nk=1327200145&s=100)

            **chen**

            April 22nd, 2019 at 12:47 pm

            **[@Rat's](#comment-14437)**

            好的, 谢谢了

            [回复](https://www.moerats.com/archives/931/? replyTo=14448#respond-post-931)



32.   ! [](https://cdn.v2ex.com/gravatar/4d8fd61c3c2795322ca45e33a71690b8? s=65&r= G&d=)

    **西瓜**

    April 22nd, 2019 at 10:56 am

    博主, 请问多个微信账号登录的是同一个机器人吗? 是不是需要两个都登录才能正常运行呢

    [回复](https://www.moerats.com/archives/931/? replyTo=14435#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        April 22nd, 2019 at 12:21 pm

        **[@西瓜](#comment-14435)**

        同一个机器人收发不同微信的消息.

        [回复](https://www.moerats.com/archives/931/? replyTo=14438#respond-post-931)

        1.   ! [](https://cdn.v2ex.com/gravatar/4d8fd61c3c2795322ca45e33a71690b8? s=65&r= G&d=)

            **西瓜**

            April 22nd, 2019 at 05:16 pm

            **[@Rat's](#comment-14438)**

            2019-04-22 17:13:59,324 \[Level 99\]: ehforwarderbot.\_\_main\_\_ (\_\_main\_\_.init; \_\_main\_\_.py:96)

            Slave channel WeChat Slave (blueset.wechat) # xigua is initialized.

            2019-04-22 17:13:59,325 \[Level 99\]: ehforwarderbot.\_\_main\_\_ (\_\_main\_\_.init; \_\_main\_\_.py:99)

            Initializing master blueset.telegram...

            2019-04-22 17:13:59,871 \[Level 99\]: ehforwarderbot.\_\_main\_\_ (\_\_main\_\_.init; \_\_main\_\_.py:109)

            Master channel Telegram Master (blueset.telegram) # Default profile is initialized.

            2019-04-22 17:13:59,872 \[Level 99\]: ehforwarderbot.\_\_main\_\_ (\_\_main\_\_.init; \_\_main\_\_.py:111)

            All channels initialized.

            2019-04-22 17:13:59,872 \[Level 99\]: ehforwarderbot.\_\_main\_\_ (\_\_main\_\_.init; \_\_main\_\_.py:125)

            All middlewares are initialized.

            [回复](https://www.moerats.com/archives/931/? replyTo=14453#respond-post-931)

            1.   ! [](https://cdn.v2ex.com/gravatar/4d8fd61c3c2795322ca45e33a71690b8? s=65&r= G&d=)

                **西瓜**

                April 22nd, 2019 at 07:03 pm

                **[@西瓜](#comment-14453)**

                Exception in thread Thread-2:
                Traceback (most recent call last):
                File "/usr/lib/python3.6/threading.py", line 916, in \_bootstrap\_inner

                self.run()

                File "/usr/lib/python3.6/threading.py", line 864, in run

                self.\_target(\*self.\_args, \*\*self.\_kwargs)

                File "/usr/local/lib/python3.6/dist-packages/itchat/components/login.py", line 281, in maintain\_loop

                exitCallback()

                File "/usr/local/lib/python3.6/dist-packages/efb\_wechat\_slave/\_\_init\_\_.py", line 264, in exit\_callback

                if not getattr(coordinator, 'master', default= None):

                TypeError: getattr() takes no keyword arguments

                2019-04-22 17:14:23,301 \[ERROR\]: telegram.ext.dispatcher (dispatcher.process\_update; dispatcher.py:301)

                An uncaught error was raised while processing the update

                Traceback (most recent call last):
                File "/usr/local/lib/python3.6/dist-packages/telegram/ext/dispatcher.py", line 279, in process\_update

                handler.handle\_update(update, self)

                File "/usr/local/lib/python3.6/dist-packages/telegram/ext/commandhandler.py", line 173, in handle\_update

                return self.callback(dispatcher.bot, update, \*\*optional\_args)

                File "/usr/local/lib/python3.6/dist-packages/efb\_telegram\_master/commands.py", line 132, in extra\_listing

                msg += "\\n\\n<b>%s %s (%s) </b>" % (i.channel\_emoji, i.module\_name, i.module\_id)

                AttributeError: 'WeChatChannel' object has no attribute 'module\_name'
                ^CException ignored in: <module 'threading' from '/usr/lib/python3.6/threading.py'>
                Traceback (most recent call last):
                File "/usr/lib/python3.6/threading.py", line 1294, in \_shutdown

                t.join()

                File "/usr/lib/python3.6/threading.py", line 1056, in join

                self.\_wait\_for\_tstate\_lock()

                File "/usr/lib/python3.6/threading.py", line 1072, in \_wait\_for\_tstate\_lock

                elif lock.acquire(block, timeout):

                [回复](https://www.moerats.com/archives/931/? replyTo=14456#respond-post-931)

                1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

                    **[Rat's](http://www.moerats.com/)**博主

                    April 23rd, 2019 at 12:23 am

                    **[@西瓜](#comment-14456)**

                    这个问题和你说的一样吧? https://github.com/blueset/efb-telegram-master/issues/51

                    [回复](https://www.moerats.com/archives/931/? replyTo=14465#respond-post-931)

                    1.   ! [](https://cdn.v2ex.com/gravatar/4d8fd61c3c2795322ca45e33a71690b8? s=65&r= G&d=)

                        **西瓜**

                        April 23rd, 2019 at 11:22 am

                        **[@Rat's](#comment-14465)**

                        不是, 主要是我没办法登录两个微信. 请问通过用 2 个机器人能不能解决这个问题呢

                        [回复](https://www.moerats.com/archives/931/? replyTo=14468#respond-post-931)

                        1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

                            **[Rat's](http://www.moerats.com/)**博主

                            April 23rd, 2019 at 01:43 pm

                            **[@西瓜](#comment-14468)**

                            可以试试, 不过建议一个手动, 一个 docker, 或者 2 个 docker, 不然可能会有冲突

                            [回复](https://www.moerats.com/archives/931/? replyTo=14472#respond-post-931)

                            1.   ! [](https://cdn.v2ex.com/gravatar/4d8fd61c3c2795322ca45e33a71690b8? s=65&r= G&d=)

                                **西瓜**

                                April 23rd, 2019 at 10:21 pm

                                **[@Rat's](#comment-14472)**

                                是不是可以用两个机器人啊? 不可能一个 VPS 只能装一个机器人吧

                                [回复](https://www.moerats.com/archives/931/? replyTo=14483#respond-post-931)






        2.   ! [](https://cdn.v2ex.com/gravatar/4d8fd61c3c2795322ca45e33a71690b8? s=65&r= G&d=)

            **西瓜**

            April 22nd, 2019 at 05:12 pm

            **[@Rat's](#comment-14438)**

            我按照给的方法, 其中一个加了#+ ID, 确实可以出现两个二维码. 但是第二个二维码扫码登录虽然显示登录成功, 但是实际上用不了.
            不知道哪里错了

            [回复](https://www.moerats.com/archives/931/? replyTo=14452#respond-post-931)



33.   ! [](https://cdn.v2ex.com/gravatar/d219af79b45e5891507fda4c4c2139a0? s=65&r= G&d=)

    **[repostone](https://repostone.home.blog/)**

    April 22nd, 2019 at 09:19 am

    路过, 看看.

    [回复](https://www.moerats.com/archives/931/? replyTo=14434#respond-post-931)

34.   ! [](https://cdn.v2ex.com/gravatar/0f5d9f135a12c694cd9562ccb2d9b2e8? s=65&r= G&d=)

    **[阿三](https://www.moerats.com/)**

    April 21st, 2019 at 06:20 am

    博主你好, 我在 Telegram 群 Userid 的获取那里出了点问题, 我在浏览器输入好地址并回车以后显示 {"ok": true,"result":\[\]}
    请问是哪里出了问题呢?

    [回复](https://www.moerats.com/archives/931/? replyTo=14392#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        April 21st, 2019 at 09:39 am

        **[@阿三](#comment-14392)**

        需要你先给机器人发个消息, 然后通过 api 查看才有数据

        [回复](https://www.moerats.com/archives/931/? replyTo=14399#respond-post-931)

        1.   ! [](https://cdn.v2ex.com/gravatar/0f5d9f135a12c694cd9562ccb2d9b2e8? s=65&r= G&d=)

            **[阿三](https://www.moerats.com/)**

            April 21st, 2019 at 02:01 pm

            **[@Rat's](#comment-14399)**

            感谢博主, 成功啦! 请问二维码要保存下来吗? 还有个小问题我在使用 Ctrl+ C 断开运行后, 用 screen -dmS EHF ehforwarderbot 这条命令不能后台运行, 我用 ls 查看显示没在运行

            [回复](https://www.moerats.com/archives/931/? replyTo=14402#respond-post-931)

            1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

                **[Rat's](http://www.moerats.com/)**博主

                April 21st, 2019 at 03:14 pm

                **[@阿三](#comment-14402)**

                不需要保存, 下次登录过期后会重新给二维码你登录, 至于后面的, 你用 screen -ls 就可以查看当前会话列表了, 一般是没问题的.

                [回复](https://www.moerats.com/archives/931/? replyTo=14404#respond-post-931)

                1.   ! [](https://cdn.v2ex.com/gravatar/0f5d9f135a12c694cd9562ccb2d9b2e8? s=65&r= G&d=)

                    **[阿三](https://www.moerats.com/)**

                    April 21st, 2019 at 03:43 pm

                    **[@Rat's](#comment-14404)**

                    谢谢, 已经弄好了. 打算晚上也把 QQ 的弄了. 博主, 我找了一下你的博客没有打赏功能吗? 在你这里学习了很多想交一下学费

                    [回复](https://www.moerats.com/archives/931/? replyTo=14406#respond-post-931)

                    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

                        **[Rat's](http://www.moerats.com/)**博主

                        April 21st, 2019 at 06:14 pm

                        **[@阿三](#comment-14406)**

                        ! [](//www.moerats.com/usr/paopao/ 滑稽.png) 你不说我都不知道还有这个功能, 已经开了打赏功能.

                        [回复](https://www.moerats.com/archives/931/? replyTo=14409#respond-post-931)






35.   ! [](https://q.qlogo.cn/g? b= qq&nk=3285273699&s=100)

    **lexo**

    April 20th, 2019 at 01:39 pm

    博主, 之前用 centos7 没问题, 刚刚用 ubuntu 出现 error: invalid command 'bdist\_wheel', 咋解决! [](//www.moerats.com/usr/paopao/ 乖.png)

    [回复](https://www.moerats.com/archives/931/? replyTo=14373#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        April 20th, 2019 at 02:18 pm

        **[@lexo](#comment-14373)**

        这个问题运行下 pip3 install wheel 命令应该就可以好了

        [回复](https://www.moerats.com/archives/931/? replyTo=14375#respond-post-931)

        1.   ! [](https://q.qlogo.cn/g? b= qq&nk=3285273699&s=100)

            **lexo**

            April 21st, 2019 at 03:10 pm

            **[@Rat's](#comment-14375)**

            谢谢博主, 解决了!! [](//www.moerats.com/usr/paopao/ 花心.png)

            [回复](https://www.moerats.com/archives/931/? replyTo=14403#respond-post-931)

        2.   ! [](https://cdn.v2ex.com/gravatar/a5876434c2932c5c88c0866ddd6b3465? s=65&r= G&d=)

            **[小俊](https://www.xjdog.cn)**

            April 20th, 2019 at 03:40 pm

            **[@Rat's](#comment-14375)**

            博主, 我这个问题怎么解决呀? 另外一篇文章, 可以先加个联系方式吗?! [](//www.moerats.com/usr/paopao/ 呵呵.png)

            [回复](https://www.moerats.com/archives/931/? replyTo=14378#respond-post-931)

            1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

                **[Rat's](http://www.moerats.com/)**博主

                April 21st, 2019 at 12:26 am

                **[@小俊](#comment-14378)**

                那个问题已经提示你了.

                [回复](https://www.moerats.com/archives/931/? replyTo=14391#respond-post-931)




36.   ! [](https://cdn.v2ex.com/gravatar/f411333bb368ceaf818f5508ff52d8f8? s=65&r= G&d=)

    **lala**

    April 20th, 2019 at 12:18 pm

    用这个回复 qq 微信消息是真的舒服

    [回复](https://www.moerats.com/archives/931/? replyTo=14356#respond-post-931)

    1.   ! [](https://q.qlogo.cn/g? b= qq&nk=903323244&s=100)

        **pppppplh**

        June 16th, 2020 at 09:11 pm

        **[@lala](#comment-14356)**

        您好 您 QQ 的也弄好了吗 我 tele 一直收不到消息 但网页登录后可以看到日志

        [回复](https://www.moerats.com/archives/931/? replyTo=29482#respond-post-931)

    2.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        April 20th, 2019 at 12:34 pm

        **[@lala](#comment-14356)**

        ! [](//www.moerats.com/usr/paopao/ 滑稽.png) 感觉没有自带客户端方便, 除非需要同时收发多个账户, 就方便很多.

        [回复](https://www.moerats.com/archives/931/? replyTo=14359#respond-post-931)

        1.   ! [](https://cdn.v2ex.com/gravatar/f411333bb368ceaf818f5508ff52d8f8? s=65&r= G&d=)

            **lala**

            April 20th, 2019 at 02:01 pm

            **[@Rat's](#comment-14359)**

            主要是我不想打开 qq 微信,! [](//www.moerats.com/usr/paopao/ 滑稽.png)

            [回复](https://www.moerats.com/archives/931/? replyTo=14374#respond-post-931)



37.   ! [](https://cdn.v2ex.com/gravatar/f411333bb368ceaf818f5508ff52d8f8? s=65&r= G&d=)

    **lala**

    April 20th, 2019 at 12:08 pm

    出来了, 给力博主. 两种方法很详细

    [回复](https://www.moerats.com/archives/931/? replyTo=14355#respond-post-931)

38.   ! [](https://cdn.v2ex.com/gravatar/1260f5e559e7d7b90024884a781cd7e7? s=65&r= G&d=)

    **小白**

    April 20th, 2019 at 11:31 am

    之前曾经断断续续搞过, 但是成功率为 0, 一次也没有成功过,
    各种条件不满足, 感觉这种东西目前还是 alfa 阶段, 瞎搞的东西, 各种报错

    [回复](https://www.moerats.com/archives/931/? replyTo=14351#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        April 20th, 2019 at 12:29 pm

        **[@小白](#comment-14351)**

        现在已经是 V2 版本了, 我用的时候还行, 一切正常

        [回复](https://www.moerats.com/archives/931/? replyTo=14357#respond-post-931)

    2.   ! [](https://cdn.v2ex.com/gravatar/1260f5e559e7d7b90024884a781cd7e7? s=65&r= G&d=)

        **小白**

        April 20th, 2019 at 11:49 am

        **[@小白](#comment-14351)**

        peewee.ImproperlyConfigured: SQLite driver not installed

        [回复](https://www.moerats.com/archives/931/? replyTo=14353#respond-post-931)

        1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

            **[Rat's](http://www.moerats.com/)**博主

            April 20th, 2019 at 12:33 pm

            **[@小白](#comment-14353)**

            什么系统? 或者你先试试这个命令 pip3 install pysqlite3

            [回复](https://www.moerats.com/archives/931/? replyTo=14358#respond-post-931)

            1.   ! [](https://cdn.v2ex.com/gravatar/1260f5e559e7d7b90024884a781cd7e7? s=65&r= G&d=)

                **小白**

                April 20th, 2019 at 10:14 pm

                **[@Rat's](#comment-14358)**

                换 debian8 成功了, 多谢博主

                [回复](https://www.moerats.com/archives/931/? replyTo=14385#respond-post-931)




39.   ! [](https://cdn.v2ex.com/gravatar/eb921054682fb81ae73de51416522067? s=65&r= G&d=)

    **map**

    April 19th, 2019 at 07:29 pm

    大佬什么程序或者软件可以转发 QQ 消息去微信?

    [回复](https://www.moerats.com/archives/931/? replyTo=14330#respond-post-931)

    1.   ! [](https://cdn.v2ex.com/gravatar/74fb0e4834eb666f5026da78ddb089b3? s=65&r= G&d=)

        **[Rat's](http://www.moerats.com/)**博主

        April 19th, 2019 at 10:45 pm

        **[@map](#comment-14330)**

        这个不太清楚.

        [回复](https://www.moerats.com/archives/931/? replyTo=14336#respond-post-931)


40.   ! [](https://cdn.v2ex.com/gravatar/7311e099b43f2d572b6197235e4818bb? s=65&r= G&d=)

    **[半吊子的静树](https://www.yooomu.com)**

    April 19th, 2019 at 11:31 am

    有点意思, 搞个玩玩! [](//www.moerats.com/usr/paopao/ 阴险.png)

    [回复](https://www.moerats.com/archives/931/? replyTo=14321#respond-post-931)
