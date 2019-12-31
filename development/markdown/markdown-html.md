---
title: Markdown 中使用 html
date: 2019-12-24 14:00:00
tags: '开发技能'
categories:
  - ['markdown']
permalink: markdown-html
---

# <i class="fa fa-bomb" aria-hidden="true"></i> 缘由

本着觉得一个网站不能孤零零的就一个首页和归档的菜单, 于是把 next 中的大部分菜单都启用了, 这样就多了 about, sitemap 还有个公益的菜单.

看着都挺好的, 可是 about 是个单页面, 里面需要自己写页面, 可惜自己本来想象空间就匮乏, 面对一整个白板, 还要用 markdown 这种填满空白. :dizzy_face::dizzy_face::dizzy_face:

<!-- more -->

# :zap::zap:

想想每次自己做一些页面都是去 copy 的 html + css, markdown 其实也是简单的 html, 里面还是可以用 html 来表示, 只是写法比较简陋而已.

总的来说, 可以直接在文档总使用 html 带 style 这种来表示页面

## 使用

```html
<dl>
  <dt style="color: green;">这是一个 dt 标签</dt>
  <dd>这是一个 dd 标签</dd>

  <dt style="color: blue;">这是一个 dt 标签</dt>
  <dd>这是一个 dd 标签</dd>
</dl>
```

实际会显示成如下所示

<dl>
  <dt style="color: green;">这是一个 dt 标签</dt>
  <dd>这是一个 dd 标签</dd>

  <dt style="color: blue;">这是一个 dt 标签</dt>
  <dd>这是一个 dd 标签</dd>
</dl>

## 注意事项

* 块元素例如 `<div></div>`, `<table></table>`, `<pre></pre>`, `<p></p>` 等标签, 需要在前后加上空行与其他内容隔开, 还要求它们的开始标签和结尾标签不能用制表符和空格来缩进.
* 行内元素如 `<span></span>`, `<cite></cite>`, `<del></del>` 可以在 markdown 的段落, 列表或者标题里面随意使用.

## 其他元素

Hexo 默认是用 Font Awesome 开作为图标的, 看了一下生成的页面都包含了相关资源的引用.

```html
<head>
  <link rel="stylesheet" href="/lib/font-awesome/css/font-awesome.min.css">
</head>
```

这样我们可以在 markdown 文件里面直接用 `<i></i>` 标签来插入图标了. eg. <i class="fa fa-spin fa-bug" aria-hidden="true"></i> <i class="fa fa-spin fa-refresh" aria-hidden="true"></i>

# 总结

* 优点:
  * 页面个性化, 可以有更多的排版
  * 可以拥有特殊的图标
* 缺点:
  * 复杂化, 失去了 markdown 的初衷
  * GitHub, 本地编辑器, 还有最终渲染出来的展示不一致

总的来说, 能有多种方法来进行表达是一种好的途径, 但是沉迷方法而脱离了写作的目的这就不好了. 当然个人还是觉得能不用就不用, 特别是在技术文章中, 能用代码表示的就不用文字, 能用文字的就不用图片, 毕竟能写出大家都看懂的东西才是好的. 能用文字表述好一种场景也是一个不错的技能.
