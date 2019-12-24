---
title: HTTPS 简介
date: 2019-12-21 18:28:54
tags: '技术'
categories:
  - ['使用说明', '技术']
permalink: https-introduction
---

# HTTPS

## 概念

1. HTTP 协议 (HyperText Transfer Protocol, 超文本传输协议): 是客户端浏览器或其他程序与 Web 服务器之间的应用层通信协议.
2. HTTPS 协议 (HyperText Transfer Protocol over Secure Socket Layer): 可以理解为 HTTP+SSL/TLS,  即 HTTP 下加入 SSL 层, HTTPS 的安全基础是 SSL, 因此加密的详细内容就需要 SSL, 用于安全的 HTTP 数据传输.
3. SSL (Secure Socket Layer, 安全套接字层): 1994 年为 Netscape 所研发, SSL 协议位于 TCP/IP 协议与各种应用层协议之间, 为数据通讯提供安全支持.
4. TLS (Transport Layer Security, 传输层安全): 其前身是 SSL, 它最初的几个版本 (SSL 1.0、SSL 2.0、SSL 3.0) 由网景公司开发, 1999 年从 3.1 开始被 IETF 标准化并改名, 发展至今已经有 TLS 1.0、TLS 1.1、TLS 1.2 三个版本. SSL3.0 和 TLS1.0 由于存在安全漏洞, 已经很少被使用到. TLS 1.3 改动会比较大, 目前还在草案阶段, 目前使用最广泛的是 TLS 1.1、TLS 1.2.

<!-- more -->

## 过程解析

### 三次 tcp 握手

1. 主机 A 发送位码为 syn＝1,随机产生 seq number=0 的数据包到服务器, 主机 B 由 SYN=1 知道, A 要求建立联机;
2. 主机 B 收到请求后要确认联机信息, 向 A 发送 ack number=(主机 A 的 seq+1),syn=1,ack=1,随机产生 seq=0 的包
3. 主机 A 收到后检查 ack number 是否正确, 即第一次发送的 seq number+1,以及位码 ack 是否为 1, 若正确, 主机 A 会再发送 ack number=(主机 B 的 seq+1),ack=1, 主机 B 收到后确认 seq 值与 ack=1 则连接建立成功.

![tcp 三次握手](https-introduction/tcp)

完成三次握手, 主机 A 与主机 B 开始传送数据. 注意, seq number 是随机数, 这里的数字显示为相对数字, 方便对照看.

### 加密

1. 对称加密: 加密和解密都用一个密钥, 有点是速度快. 缺点吗, 显而易见的是 双方要用这个通信的话 必须得把密钥告知对方,  但是这个密钥一旦被截获了, 就可以随便 decode 出来明文. 实际上不够安全.
2. 非对称加密: 有一对密钥, 即有公钥也有私钥. 公钥随便发, 私钥不会发出去 各自保管. 用一个加密的话, 解密就只能用另外一个. 比如你向银行请求一个公钥, 银行把公钥发给你, 你用公钥加密一条信息以后再发给银行, 然后银行用私钥解密信息. 这个过程就算有人拿到公钥, 但是因为非对称加密用一个加密 只能用另外一个解密, 所以拿到这公钥也没用, 因为无法使用公钥解密, 能解密的私钥还在银行那里, 银行当然不会传出去, 所以非对称加密很安全. 但是这种非对称加密非常消耗资源, 速度极慢, 所以要有限使用. 不能无节制使用.
3. HASH 加密算法: 这个就好像 MD5 这样的加密方式, 是不可逆的.

### 确保安全通信的三个原则

1. 数据内容的加密
2. 通讯双方的身份校验
3. 数据内容的完整性

### https 的设计思路

1. 服务器将自己的证书发送给客户端;
1. 客户端通过层层 CA 验证证书是真的, 从证书里面拿出来服务器的公钥;
1. 客户端通过服务器公钥将一个随机数发送给服务器;
1. 服务器通过自己的私钥解密得到随机数, 这样客户端和服务器都能通过这个随机出算出一个对称秘钥;
1. 之后双方通过对称秘钥加密数据进行通信.

简单的过程是这样, 通过非对称加密的方式来传输一个对称加密的秘钥, 最终通过对称秘钥进行数据加密, 这样能保证传输的效率.

这里关键的一步就是服务器发送过来的证书, 是通过一个信任的 CA 签名的, 所以值得信任. 公钥即使被中间人截获以后也没用, 因为拿到公钥也解密不出来到底双方是用哪种算法加密的.

![中间人攻击](https-introduction/man-in-middle)

### 解决中间人攻击问题

1. 使用权威的第三方机构也就是 CA 向安全的服务器颁发证书. 来证明这台服务器的合法性.
2. 服务器通过这个证书来把自己的公钥加密以后发给客户端
3. 客户端收到这个加密后的公钥以后 , 就用第三方机构的公钥 把这个服务器返回的加密后的公钥 解密 从而得到真正的服务器的公钥

操作系统或者浏览器自身就带有权威机构的第三方公钥. 如果中间人得到 CA 认证怎么办？这种情况基本没办法处理, 如果发生, 那么这个 CA 下面所有的证书都被认为非法了.

### HTTPS 的流程

![HTTPS 的流程](https-introduction/https)

1. 先看蓝色的部分, 可以看出来, 这是 tcp 链接. 所以 https 的加密层也是在 tcp 之上的.
1. 客户端首先发起 clientHello 消息. 包含一个客户端随机生成的 random1 数字, 客户端支持的加密算法, 以及 SSL 信息.
1. 服务器收到客户端的 clientHello 消息以后, 取出客户端法发来的 random1 数字, 并且取出客户端发来的支持的加密算法, 然后选出一个加密算法, 并生成一个随机数 random2, 发送给客户端 serverhello
1. 让客户端对服务器进行身份校验,服务端通过将自己的公钥通过数字证书的方式发送给客户端
1. 客户端收到服务端传来的证书后, 先从 CA 验证该证书的合法性, 验证通过后取出证书中的服务端公钥, 再生成一个随机数 Random3, 再用服务端公钥非对称加密 Random3 生成 PreMaster Key. 并将 PreMaster Key 发送到服务端,服务端通过私钥将 PreMaster Key 解密获取到 Random3,此时客户端和服务器都持有三个随机数 Random1 Random2 Random3,双方在通过这三个随即书生成一个对称加密的密钥.双方根据这三个随即数经过相同的算法生成一个密钥,而以后应用层传输的数据都使用这套密钥进行加密.
1. Change Cipher Spec:告诉客户端以后的通讯都使用这一套密钥来进行.
1. 最后 ApplicationData 全部使用对称加密的原因就是非对称加密太卡, 对称加密不影响性能. 所以实际上也看的出来 HTTPS 的真正目的就是保证对称加密的 密钥不被破解, 不被替换, 不被中间人攻击, 如果发生了上述情况, 那么 HTTPS 的加密层也能获知, 避免发生事故.

### https 抓包

Charles 作为一个中间人代理, 当浏览器和服务器通信时, Charles 接收服务器的证书, 但动态生成一张证书发送给浏览器, 也就是说 Charles 作为中间代理在浏览器和服务器之间通信, 所以通信的数据可以被 Charles 拦截并解密. 由于 Charles 更改了证书, 浏览器校验不通过会给出安全警告, 必须安装 Charles 的证书后才能进行正常访问.

使用抓包工具的第一步就是在你自己设备中信任 Charles 的 CA 证书, 在自己的设备中添加了一个 CA, 请求的时候, Charles 通过自己的 CA 签名了一个自己的公钥, 发送给客户端, 客户端就误以为是服务器了, 这样之后的流程都会先走到 Charles 然后才会走到目标服务器.

Charles 扮演了一个中间人的角色, 而且这个中间人是我们自己设置的. 用户本身主动添加中间人, 跟中间人攻击完全是两码事, 一个正常用户, 设备中 CA 无异常, 中间人根本参与不进来, 即便从网络传输过程中获取了 https 的数据包, 也无法解密, 获取不了数据, 更不要谈篡改了.

### ssl-pinning

证书一块打包到客户端里. 这样在 HTTPS 建立时与服务端返回的证书比对一致性, 进而识别出中间人攻击后直接在客户端侧中止连接. ssl-pinning 技术在 AFNetworking 中已经得到支持, 参照这篇[文章](http://nelson.logdown.com/posts/2015/04/29/how-to-properly-setup-afnetworking-security-connection/)

### 自签名证书

证书的三个作用 加密通信和身份验证 (验证对方确实是对方声称的对象) 和数据完整性 (无法被修改, 修改了会被知)

x509 的证书编码格式有两种

1. PEM(Privacy-enhanced Electronic Mail) 是明文格式的 以 `-----BEGIN CERTIFICATE-----` 开头, 已 `-----END CERTIFICATE-----` 结尾, 中间是经过 base64 编码的内容,apache 需要的证书就是这类编码的证书 查看这类证书的信息的命令为: `openssl x509 -noout -text -in server.pem` 其实 PEM 就是把 DER 的内容进行了一次 base64 编码
2. DER 是二进制格式的证书  查看这类证书的信息的命令为: `openssl x509 -noout -text -inform der -in server.der`

扩展名:

1. `.crt` 证书文件 , 可以是 DER (二进制) 编码的, 也可以是 PEM ( ASCII (Base64) ) 编码的 , 在类 unix 系统中比较常见
1. `.cer` 也是证书 常见于 Windows 系统 编码类型同样可以是 DER 或者 PEM 的, windows 下有工具可以转换 crt 到 cer
1. `.csr` 证书签名请求  一般是生成请求以后发送给 CA, 然后 CA 会给你签名并发回证书
1. `.key` 一般公钥或者密钥都会用这种扩展名, 可以是 DER 编码的或者是 PEM 编码的 查看 DER 编码的 (公钥或者密钥) 的文件的命令为 `openssl rsa -inform DER -noout -text -in xxx.key` 查看 PEM 编码的 (公钥或者密钥) 的文件的命令为 `openssl rsa -inform PEM  -noout -text -in xxx.key`
1. `.p12` 证书 包含一个 X509 证书和一个被密码保护的私钥

使用 openssl 生成证书
1. 生成私钥 `openssl genrsa -des3 -out rootCA.key 2048`
1. 根证书 pem `openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.pem`
1. 创建一个新的 OpenSSL 配置文件, `server.csr.cnf`

```
[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn

[dn]
C=CN
ST=shenzhen
L=shenzhen
O=shenzhen
OU=shenzhen
emailAddress=admin@localhost
CN=localhost
```

1. 创建一个 v3.ext 文件, 以创建一个 X509 v3 证书. 注意我们指定了 subjectAltName 选项.

```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = localhost
```

1. 创建证书密钥 server.key; 以存储有 localhost 的配置 server.csr.cnf 进行设置. `openssl req -new -sha256 -nodes -out server.csr -newkey rsa: 2048 -keyout server.key -config < (cat server.csr.cnf) `
1. 证书签名请求通过我们之前创建的根证书 rootCA.pem 颁发, 创建出一个 localhost 的域名证书. 输出证书文件 server.crt. `openssl x509 -req -in server.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out server.crt -days 500 -sha256 -extfile v3.ext`
1. 将文件 `server.key` 和 `server.crt` 文件复制到服务器上可访问的位置

## 总结

![HTTPS 基本原理](https-introduction/full)

1. 一般【客户端】首先发起请求, 例如请求网站 www.thinktxt.com/, 生成一个随机数 (RandomC) , 携带支持的 TLS 版本、加密算法等信息发送至【服务端】.
2. 【服务端】收到请求, 返回证书、一个随机数 (RandomS) 、协商加密算法.
3. 【客户端】拿到证书后开始进行校验, 验证其合法性. 客户端通过本地浏览器或操作系统内置的权威第三方认证机构的 CA 证书进行验证, 一个证书包含域名、证书编号、公钥、有效期等信息, 证书编号是在服务器管理员通过第三方证书机构申请证书的时候, 第三方机构用他们的私钥对证书编号进行加密存入证书, 根据编号生成方法生成证书编号 (证书本身携带了生成证书编号的方法) , 与 CA 证书公钥解密得出的证书编号进行对比, 验证不通过或者证书过期等情况就提示存在风险 (浏览器的红色警告) , 验证通过则进行下一步.
4. 【客户端】生成一个随机数 (PreMaster Key) , 此时已经有第三个随机数了, 根据三个随机数 (RandomC、RandomS、PreMaster Key) 按照双方约定的算法生成用于后面会话的同一把的“会话密钥”.
5. 【客户端】将随机数 (PreMaster Key) 通过公钥加密后发送至【服务端】.
6. 【服务端】收到密文后用私钥进行解密, 得到随机数 (PreMaster Key) , 此时服务端也拥有了三个随机数, 根据三个随机数按照事先约定的加密算法生成用于后面会话的同一把的“会话密钥”.
7. 【服务端】计算此前所有内容的握手消息 hash 值, 并用“会话密钥”加密后发送至客户端用于验证.
8. 【客户端】解密并计算握手消息的 hash 值, 如果与服务端发来的 hash 一致, 此时握手过程结束.
9. 验证通过后, 开始正常的加密通信.

## 引用

[深入理解 HTTPS 协议](https://juejin.im/post/5a2fbe1b51882507ae25f991)
[抓包工具能获取 HTTPS 数据, 所以 HTTPS 并不安全？](https://learnku.com/articles/19238)
[有关 ssl-pinning 的总结](https://www.jianshu.com/p/22b56d977825)
[关于证书的那些事: 自签名证书和私有 CA 签名证书等](https://www.jianshu.com/p/cc6b804a4d80)
[自签名数字证书的使用](https://www.jianshu.com/p/44a3efae1d84)