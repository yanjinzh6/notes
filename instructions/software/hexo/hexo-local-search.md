---
title: Hexo 本地搜索
date: 2020-06-04 19:00:00
tags: 'Hexo'
categories:
  - ['使用说明', '软件']
permalink: hexo-local-search
mathjax: false
---

## 方法

Hexo 提供了搜索的插件 `hexo-generator-searchdb`

原理是通过生成一个 `search.xml` 的文件, 该插件将所有的文档都生成到 `<entry>` 字段里面, 具体结构如下

```xml
<search>
  <entry>
    <title>标题</title>
    <url>链接</url>
    <content>内容</content>
    <categories>
      <category>分类 1</category>
      <category>分类 n</category>
    </categories>
    <tags>
      <tag>标签 1</tag>
      <tag>标签 n</tag>
    </tags>
  </entry>
</search>
```

这样可以很方便的进行全文搜索

<!-- more -->

## 使用

### 安装插件

```sh
yarn add hexo-generator-searchdb
```

### 修改配置文件

需要修改 Hexo 根目录的 `_config.yml` 文件, 增加 `search` 字段配置

```yaml
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

next 主题需要修改主题的 `_config.yml` 文件, 将 `local_search.enable` 修改为 `true`

```yaml
# Local Search
# Dependencies: https://github.com/theme-next/hexo-generator-searchdb
local_search:
  enable: true
  # If auto, trigger search by changing input.
  # If manual, trigger search by pressing enter key or search button.
  trigger: auto
  # Show top n results per article, show all results by setting to -1
  top_n_per_article: 1
  # Unescape html strings to the readable one.
  unescape: false
  # Preload the search data when the page loads.
  preload: false
```

## 参考

- [hexo-generator-searchdb](https://github.com/theme-next/hexo-generator-searchdb)
