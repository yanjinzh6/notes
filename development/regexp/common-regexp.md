---
title: 常用正则表达式
date: 2020-02-05 23:00:00
tags: 'Regexp'
categories:
  - ['开发', 'Regexp']
permalink: common-regexp
---

# 中文

- 匹配中文: `[\u4e00-\u9fa5]`
- 匹配双字节字符 (包括汉字在内): `[^\x00-\xff]`, 可以用来计算字符串的长度 (一个双字节字符长度计 2, ASCII 字符计 1)

# 空行

匹配空白行: `\n\s*\r`

# 首尾空白字符

匹配首尾空白字符: `^\s*|\s*$`

# 中英文符号连着

`([0-9a-zA-Z\])([\u4e00-\u9fa5])`

# 引用

- [正则表达式之任意字符](https://www.w3cschool.cn/regexp/1ngu1pqi.html)