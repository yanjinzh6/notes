---
title: Jenkins 插件中心源
date: 2020-04-14 11:00:00
tags: 'Jenkins'
categories:
  - ['使用说明', 'CI']
permalink: jenkins-plugin-center-mirror
photo:
---

Jenkins 插件中心在国内访问过程是非常不愉快的, 所以也有了各种国内的镜像源了, 但由于 [Jenkins 插件](https://github.com/jenkins-infra/update-center2/pull/245)的安全机制 - 非对称加密, 官方用其中一把钥匙给文件做了签名, 并保管起来；把另外一把钥匙对外公布 (保存在发行版中).  只有通过了公钥验证的 `update-center.json` 文件, 才会被使用到.

也就是要把发行版中的密钥替换为国内的镜像源提供的密钥才能正常使用, 这个 Jenkins 中文社区已经解决了, 可以[参考一下](https://www.oschina.net/news/111266/jenkins-plugin-mirror)

如果我们要使用[清华大学的 Jenkins 镜像源](https://mirrors.tuna.tsinghua.edu.cn/jenkins/), 则需要自己手动处理一下, 因为清华大学的 [Jenkins 配置文件](https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json)里面所有插件的下载地址还是国外的地址, 所有需要手动替换, 或者使用使用代理, 只要把 `mirrors.jenkins-ci.org` 代理到 `mirrors.tuna.tsinghua.edu.cn/jenkins` 即可

当然也可以考虑自己定制 Jenkins 镜像, 这又是另一个操作了.

参考

- [Jenkins 插件中心国内镜像源发布](https://www.oschina.net/news/111266/jenkins-plugin-mirror)
- [jenkins 插件下载加速最终方案](https://www.liangzl.com/get-article-detail-144870.html)
