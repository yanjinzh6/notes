---
title: Microk8s 内置组件
date: 2020-01-28 22:30:00
tags: '配置'
categories:
  - ['部署', 'k8s']
permalink: microk8s-addones
photo:
---

> 该文档只是记录了 microk8s 下的附带功能

## Cilium

Cilium 用于透明地保护使用Linux容器管理平台（如Docker和Kubernetes）部署的应用程序服务之间的网络连接。

Cilium的基础是一种名为BPF的新Linux内核技术，它可以在Linux本身动态插入强大的安全可见性和控制逻辑。由于BPF在Linux内核中运行，因此可以应用和更新Cilium安全策略，而无需对应用程序代码或容器配置进行任何更改。

### BPF

柏克莱封包过滤器（Berkeley Packet Filter，缩写 BPF），是类Unix系统上数据链路层的一种原始接口，提供原始链路层封包的收发，除此之外，如果网卡驱动支持洪泛模式，那么它可以让网卡处于此种模式，这样可以收到网络上的所有包，不管他们的目的地是不是所在主机。参考维基百科和eBPF简史。

## ctr

## helm

## helm3

## 引用

[具备API感知的网络和安全性管理的开源软件Cilium](https://jimmysong.io/kubernetes-handbook/concepts/cilium.html)
