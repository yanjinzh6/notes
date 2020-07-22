---
title: 不间断空格
date: 2020-07-18 15:00:00
tags: 'Java'
categories:
  - ['开发', 'Java', ' 编解码']
permalink: non-breaking-space
---

## 简介

刚刚发现要替换一段文本时发现正则表达式匹配不了, 大概是这样的 `a   b`, 惊奇的发现正则表达式 `( +)` 总是无法三个空格一起匹配, 还纳闷了, 不是应该默认贪婪模式么

## 排查

正则表达式测试没问题, 那就打印一下各个字符的 ASCII 码看看, 原来这里面混杂着两种空格, ASCII 码分别是 32 和 160, 键盘输入的空格为 32, 另一个是不间断空格 (non-breaking space)

```js
' '.charCodeAt()
// 32
' '.charCodeAt()
// 160
```

```java
// 复制的空格
final char c1 = ' ';
// 手动输入的空格
final char c2 = ' ';
// 160
System.out.println((int)c1);
// 32
System.out.println((int)c2);
```

## 不间断空格

> 在用 Word 输入文章的时候, 经常会遇到一个由多个单词组成的词组被分隔在两行文字里, 这样很容易让人看不明白, 其实遇到这种情况, 可以使用不间断空格来代替普通空格使该词组保持在同一行文字里
> 在 word2003 之后的版本, 可以使用组合键 `ctrl+shift+space` 来输入不间断空格键

这个在 HTML 上表现出来的就是页面的 `&nbsp;` 所产生的空格, 刚好不间断空格的缩写就是 `nbsp`, 在 HTML 中的作用就是页面换行时不被打断

<!-- more -->

## 清理

也许数据中出现的多种空格混用就是通过上面那两个路径来的, 那么就需要清理了, 由于它无法被 `trim()` 裁剪, 也没办法被正则表达式 `\s` 匹配到, 所以 `StringUtils.isBlank()` 也无法识别, 所以我们只能通过简单的复制该字符并构造正则表达式来清理, 或者使用不间断空格的 Unicode 编码 `\u00A0` 来处理

```java
// 包含了不间断空格的字符串
String str = "aacsdfe ";
str = str.replace("\u00A0", "");
str = str.replaceAll("\\u00A0+", "");
```
