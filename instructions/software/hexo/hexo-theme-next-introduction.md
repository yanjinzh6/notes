---
title: Hexo next 主题使用
date: 2019-12-30 11:00:00
tags: 'Hexo'
categories:
  - ['使用说明', '软件']
permalink: hexo-theme-next-introduction
---

# 简介

[Next](https://theme-next.iissnan.com/) 主题应该算是最流行的一套主题了, 这里提供一个旧版本的中文文档. 很多基础配置都在文档中有详细介绍了.

主题的话, 各人有各自的喜好, 自己比较偏好大量留空的设置, 现在的内容同质化很严重, 看大部分文章的时候基本上都是从搜索引擎入口的, 这时候大量留空可以更好突出内容的中心.

<!--more-->

# 配置

## 浏览器小图标

```yml
favicon:
  small: /images/favicon-16x16-next.png
  medium: /images/favicon-32x32-next.png
  apple_touch_icon: /images/apple-touch-icon-next.png
  safari_pinned_tab: /images/logo.svg
  #android_manifest: /images/manifest.json # Web 应用程序清单
  #ms_browserconfig: /images/browserconfig.xml
```

这里的 `android_manifest` 参数提供一个 [Web 应用程序清单](https://developer.mozilla.org/zh-CN/docs/Web/Manifest)配置, 具体情况可以参考相应的文档, 当使用移动版浏览器时可以添加为主屏幕的网络应用程序.

## footer

底部的配置比较简单, 对比了一下旧版本主要添加了备案信息的配置.

## 版权申明

```yml
creative_commons:
  license: by-nc-sa
  sidebar: false    # 是否显示在 sidebar
  post: false       # 是否开启
  language:
```

## 主题样式

```yml
scheme: Muse    # 大量留白
#scheme: Mist   # 紧凑
#scheme: Pisces # 双栏
#scheme: Gemini #
```

## 菜单

```yml
menu:
  home: / || home
  about: /about/ || user
  tags: /tags/ || tags
  categories: /categories/ || th
  archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  sitemap: /sitemap.xml || sitemap
  commonweal: /404/ || heartbeat

# Enable / Disable menu icons / item badges.
menu_settings:
  icons: true   # 显示图标
  badges: false # 显示统计
```

可以开启预置好的菜单, 也可以直接手动添加菜单, 除了首页和归档, 其他默认是没有的, 需要自己创建.

添加标签页, 并且设置 `type` 为 `tags`.

```yml
hexo new page tags
```

```yml
---
title: tags
date: 2019-12-21 04:17:28
type: tags
comments: false
---
```

添加分类页, 并且设置 `type` 为 `categories`.

```yml
hexo new page categories
```

```yml
---
title: categories
date: 2019-12-21 04:17:28
type: categories
comments: false
---
```

添加 404 页, 这是一个自定义页面, 可以直接使用已有的样式, 也可以使用主流的公益页面, 这里配置为 QQ 空间的公益页面.

```yml
hexo new page 404
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>404</title>
</head>
<body>
<script type="text/javascript" src="//qzonestyle.gtimg.cn/qzone/hybrid/app/404/search_children.js" charset="utf-8" homePageUrl="/" homePageName="返回"></script>
</body>
</html>
```

about 页面基本上就自己发挥了, 最好使用 markdown + html, 这样渲染出来的页面比较简洁符合主题, 也可以让页面添加更多自定义样式.

`sitemap.xml` 文件是专门为爬虫准备的站点链接. 需要安装两个依赖.

```sh
npm install hexo-generator-sitemap --save
npm install hexo-generator-baidu-sitemap --save

hexo g
```

执行以上命令就可以生成 Google 和 baidu 的 `sitemap.xml`.

# 待更...