---
title: Hexo 配置 mathjax
date: 2020-05-04 17:00:00
tags: 'Hexo'
categories:
  - ['使用说明', '软件']
permalink: hexo-mathjax
mathjax: true
---

## 简介

Hexo 自带的 `hexo-renderer-marked` 模块是不支持 latex 解析的, 但是使用的 next 主题中可以看到 mathjax 的配置

版本号需要注意一下, 很多旧的版本相应的教程有些区别

- Hexo: 4.2.0
- next: 7.6.0

```yaml
# Math Formulas Render Support
math:
  # Default (true) will load mathjax / katex script on demand.
  # That is it only render those page which has `mathjax: true` in Front-matter.
  # If you set it to false, it will load mathjax / katex srcipt EVERY PAGE.
  per_page: true

  # hexo-renderer-pandoc (or hexo-renderer-kramed) required for full MathJax support.
  mathjax:
    enable: false
    # See: https://mhchem.github.io/MathJax-mhchem/
    mhchem: false

  # hexo-renderer-markdown-it-plus (or hexo-renderer-markdown-it with markdown-it-katex plugin) required for full Katex support.
  katex:
    enable: false
    # See: https://github.com/KaTeX/KaTeX/tree/master/contrib/copy-tex
    copy_tex: false
```

`per_page` 参数

- `true`: 仅当 `front-matter` 中存在 `mathjax: true` 时, 该页面才会被渲染
- `false`: 每篇文章都会进行渲染

推荐使用 `front-matter` 中定义 `mathjax: true` 的方式, 即仅当手工在文章头部添加 `mathjax` 标记时才会渲染, 提高速度

<!-- more -->

这里支持 mathjax 和 katex, 两者是互斥的, 根据注释提示, 看上去可以很方便使用, 下面记录下实际使用过程

## mathjax

mathjax 渲染速度稍慢，但对 latex 的支持较好

### hexo-renderer-kramed

虽然在注释中是出现在括号里的, 但是这个方式显然是最快捷方便的, 缺点是 `$$` 的渲染和把反引号中带有 `$` 范围的代码也会处理掉

```sh
npm uninstall hexo-renderer-marked --save
npm install hexo-renderer-kramed --save
```

修改配置为

```yaml
math:
  per_page: true
  mathjax:
    enable: true
    mhchem: false
  katex:
    enable: false
    copy_tex: false
```

会出现一行只能使用一个行内公式的情况

```js
// ./node_modules/kramed/lib/rules/inline.js
var inline = {
  escape: /^\\([\\`*{}\[\]()#$+\-.!_>])/,
  // ...
  em: /^\b_((?:__|[\s\S])+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
  // ...
};
```

修改为

```js
// ./node_modules/kramed/lib/rules/inline.js
var inline = {
  escape: /^\\([`*\[\]()#$+\-.!_>])/,
  // ...
  em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
  // ...
};
```

### hexo-math

默认的 `hexo-renderer-marked` 渲染模块添加 `hexo-math` 模块即可以使用, 就是 hexo-math 依赖的 hexo-inject 模块已不再维护, 而且报错, 但是可以正常渲染

```sh
ERROR [hexo-inject] Error injecting: undefined
ERROR (/note-website/node_modules/hexo-math/asset/inject.swig) [Line 2, Column 38]
  Error: Unable to call `JSON["stringify"]`, which is undefined or falsey
Template render error: (/note-website/node_modules/hexo-math/asset/inject.swig) [Line 2, Column 38]
  Error: Unable to call `JSON["stringify"]`, which is undefined or falsey
    at Object._prettifyError (/note-website/node_modules/nunjucks/src/lib.js:36:11)
    at /note-website/node_modules/nunjucks/src/environment.js:561:19
    at Template.root [as rootRenderFunc] (eval at _compile (/note-website/node_modules/nunjucks/src/environment.js:631:18), <anonymous>:21:3)
    at Template.render (/note-website/node_modules/nunjucks/src/environment.js:550:10)
    at Hexo.njkRenderer (/note-website/themes/next/scripts/renderer.js:24:27)
    at Promise.then.text (/note-website/node_modules/hexo/lib/hexo/render.js:75:22)
    at tryCatcher (/note-website/node_modules/bluebird/js/release/util.js:16:23)
    at Promise._settlePromiseFromHandler (/note-website/node_modules/bluebird/js/release/promise.js:547:31)
    at Promise._settlePromise (/note-website/node_modules/bluebird/js/release/promise.js:604:18)
    at Promise._settlePromise0 (/note-website/node_modules/bluebird/js/release/promise.js:649:10)
    at Promise._settlePromises (/note-website/node_modules/bluebird/js/release/promise.js:729:18)
    at _drainQueueStep (/note-website/node_modules/bluebird/js/release/async.js:93:12)
    at _drainQueue (/note-website/node_modules/bluebird/js/release/async.js:86:9)
    at Async._drainQueues (/note-website/node_modules/bluebird/js/release/async.js:102:5)
    at Immediate.Async.drainQueues [as _onImmediate] (/note-website/node_modules/bluebird/js/release/async.js:15:14)
    at runCallback (timers.js:705:18)
```

```
// ./node_modules/hexo-math/asset/inject.swig
<!-- Begin: Injected MathJax -->
<script type="text/x-mathjax-config">
  MathJax.Hub.Config({{ JSON.stringify(config) }});
</script>

<script type="text/x-mathjax-config">
  MathJax.Hub.Queue(function() {
    var all = MathJax.Hub.getAllJax(), i;
    for(i=0; i < all.length; i += 1) {
      all[i].SourceElement().parentNode.className += ' has-jax';
    }
  });
</script>

<script type="text/javascript" src="{{ src }}">
</script>
<!-- End: Injected MathJax -->
```

这里出现了 JSON 对象为空的问题, 由于使用 next 主题不需要改文件, 可以直接将出现错误的代码去掉即可

因为某些特殊字符不能使用, 也许是被 markdown 解析器解析掉了, 所以 `hexo-math` 插件提供了一个标签给特殊的符号使用 `% math %`

考虑到通用性最好还是不要使用

### hexo-renderer-pandoc

首先需要安装 pandoc

然后更换渲染模块

```sh
npm uninstall hexo-renderer-marked --save
npm install hexo-renderer-pandoc --save
```

## katex

katex 渲染速度更快，但仅支持 latex 的子集

### hexo-renderer-markdown-it

原本已经将默认渲染模块修改为 `hexo-renderer-markdown-it` 的可以简单的添加 `markdown-it-katex` 插件并启用即可

```sh
npm install markdown-it-katex --save
```

在 Hexo 的 `_config.ymal` 文件中修改 `hexo-renderer-markdown-it` 配置即可

```yaml
markdown:
  render:
    html: true
    xhtmlOut: false
    breaks: true
    linkify: true
    typographer: true
    quotes: '“”‘’'
  plugins:
    - markdown-it-katex
    - markdown-it-abbr
    - markdown-it-footnote
    - markdown-it-ins
    - markdown-it-sub
    - markdown-it-sup
  anchors:
    level: 2
    collisionSuffix: ''
    permalink: false
    permalinkClass: 'header-anchor'
    permalinkSide: 'left'
    permalinkSymbol: '¶'
    case: 0
    separator: ''
```

如果要简单, 可以直接使用 `hexo-renderer-markdown-it-plug` 模块

## 使用

直接在文章头部条件 `mathjax: true` 即可

简单测试 `$a = b + c$`

$a = b + c$

```
`$\pi$`
\\(E=mc^2\\)
\\(lim_{x\rightarrow \infty}\frac{1}{\sin x}\\)
\\(lim_{n\rightarrow \infty}(1+2^n+3^n)^\frac{1}{x+\sin n}\\)
```

$\pi$
\\(E=mc^2\\)
\\(lim_{x\rightarrow \infty}\frac{1}{\sin x}\\)
\\(lim_{n\rightarrow \infty}(1+2^n+3^n)^\frac{1}{x+\sin n}\\)

多行代码

```
$$\frac{\partial u}{\partial t}
= h^2 \left( \frac{\partial^2 u}{\partial x^2} +
\frac{\partial^2 u}{\partial y^2} +
\frac{\partial^2 u}{\partial z^2}\right)$$
```

$$\frac{\partial u}{\partial t}
= h^2 \left( \frac{\partial^2 u}{\partial x^2} +
\frac{\partial^2 u}{\partial y^2} +
\frac{\partial^2 u}{\partial z^2}\right)$$

```
$$
\left[
    \begin{array}{cc|c}
      1&2&3\newline
      4&5&6
    \end{array}
\right]
$$
```

$$
\left[
    \begin{array}{cc|c}
      1&2&3\newline
      4&5&6
    \end{array}
\right]
$$

矩阵

```
$$
A = \begin{bmatrix}
        a_{11}    & a_{12}    & ...    & a_{1n}\\
        a_{21}    & a_{22}    & ...    & a_{2n}\\
        a_{31}    & a_{22}    & ...    & a_{3n}\\
        \vdots    & \vdots    & \ddots & \vdots\\
        a_{n1}    & a_{n2}    & ... & a_{nn}\\
    \end{bmatrix} , b = \begin{bmatrix}
        b_{1}  \\
        b_{2}  \\
        b_{3}  \\
        \vdots \\
        b_{n}  \\
    \end{bmatrix}
$$
```

$$
A = \begin{bmatrix}
        a_{11}    & a_{12}    & ...    & a_{1n}\\
        a_{21}    & a_{22}    & ...    & a_{2n}\\
        a_{31}    & a_{22}    & ...    & a_{3n}\\
        \vdots    & \vdots    & \ddots & \vdots\\
        a_{n1}    & a_{n2}    & ... & a_{nn}\\
    \end{bmatrix} , b = \begin{bmatrix}
        b_{1}  \\
        b_{2}  \\
        b_{3}  \\
        \vdots \\
        b_{n}  \\
    \end{bmatrix}
$$

概率界的贝叶斯公式

```
$$
P(A_i \mid B) = \frac{P(B\mid A)P(A_i)}{\sum_{j=1}^{n}P(A_j)P(B \mid A_j)}
$$
```

$$
P(A_i \mid B) = \frac{P(B\mid A)P(A_i)}{\sum_{j=1}^{n}P(A_j)P(B \mid A_j)}
$$

```
$$
  \begin{split}
  \frac{\partial{\mathcal{E}}}{\partial{x_l}} & =
  \frac{\partial{\mathcal{E}}}{\partial{x_L}}\frac{\partial{x_L}}{\partial{x_l}}\\\\
  & = \frac{\partial{\mathcal{E}}}{\partial{x_L}}\Big(1+\frac{\partial{}}{\partial{x_l}}\sum_{i=l}^{L-1}
  \mathcal{F}(x_i,\mathcal{W}_i)\Big)
  \end{split}
$$
```

$$
  \begin{split}
  \frac{\partial{\mathcal{E}}}{\partial{x_l}} & =
  \frac{\partial{\mathcal{E}}}{\partial{x_L}}\frac{\partial{x_L}}{\partial{x_l}}\\\\
  & = \frac{\partial{\mathcal{E}}}{\partial{x_L}}\Big(1+\frac{\partial{}}{\partial{x_l}}\sum_{i=l}^{L-1}
  \mathcal{F}(x_i,\mathcal{W}_i)\Big)
  \end{split}
$$

## 部分引用来源

- [Hexo Next 主题渲染 Latex 公式的配置方法](https://roro4ever.github.io/2019/12/01/hexo-Next%E4%B8%BB%E9%A2%98%E6%B8%B2%E6%9F%93-latex-%E5%85%AC%E5%BC%8F%E7%9A%84%E9%85%8D%E7%BD%AE%E6%96%B9%E6%B3%95/hexo-next%E4%B8%BB%E9%A2%98%E6%B8%B2%E6%9F%93-latex-%E5%85%AC%E5%BC%8F%E7%9A%84%E9%85%8D%E7%BD%AE%E6%96%B9%E6%B3%95/)
- [hexo 使用 hexo-math 插件支持 MathJax](https://introspelliam.github.io/2018/03/27/hexo/hexo%E4%BD%BF%E7%94%A8hexo-math%E6%8F%92%E4%BB%B6%E6%94%AF%E6%8C%81MathJax/)
- [让 Hexo 搭建的博客支持 LaTeX](https://cps.ninja/2019/03/16/hexo-with-latex/)
