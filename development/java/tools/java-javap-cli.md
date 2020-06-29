---
title: Java javap 工具使用
date: 2020-06-14 21:00:00
tags: 'Java'
categories:
  - ['开发', 'Java', '工具']
permalink: java-javap-cli
---

## 简介

javap 是 jdk 自带的反解析工具, 根据 class 字节码文件，反解析出当前类对应的 code 区 (汇编指令) , 本地变量表, 异常表和代码行偏移量映射表, 常量池等等信息

有些信息 (如本地变量表, 指令和代码行偏移量映射表, 常量池中方法的参数名称等等) 需要在使用 javac 编译成 class 文件时，指定参数才能输出，使用 `javac -g xx.java` 就可以生成所有相关信息了, **eclipse 在默认情况编译时会帮你生成局部变量表, 指令和代码行偏移量映射表等信息**

<!-- more -->

## 用法

```sh
javap <options> <classes>

options:
  -help  --help  -?        输出此用法消息
  -version                 版本信息，其实是当前javap所在jdk的版本信息，不是class在哪个jdk下生成的,
  -v  -verbose             输出附加信息 (包括行号, 本地变量表，反汇编等详细信息)
  -l                       输出行号和本地变量表
  -public                  仅显示公共类和成员
  -protected               显示受保护的/公共类和成员
  -package                 显示程序包/受保护的/公共类 和成员 (默认)
  -p  -private             显示所有类和成员
  -c                       对代码进行反汇编
  -s                       输出内部类型签名
  -sysinfo                 显示正在处理的类的系统信息 (路径, 大小, 日期, MD5 散列)
  -constants               显示静态最终常量
  -classpath <path>        指定查找用户类文件的位置
  -bootclasspath <path>    覆盖引导类文件的位置
```

需要了解汇编代码的可以参考[官方文档](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html)

## 小结

使用 javap 命令可以查看类反汇编的常量池, 变量表, 指令代码行号表等信息, 最近排查问题就发现了有个被引用的类修改了方法参数类型, 但是偷懒没有整个项目从新编译部署, 导致提示没有相应方法的问题, 这时候通过 javap 命令比较下服务器的文件和仓库文件的结果就知道了
