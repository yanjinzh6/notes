---
title: Hexo 配置 mermaid
date: 2020-04-20 17:00:00
tags: 'Hexo'
categories:
  - ['使用说明', '软件']
permalink: hexo-mermaid
---

安装 [npm 模块](https://www.npmjs.com/package/hexo-filter-mermaid-diagrams)

修改 Hexo 配置文件 `_config.yml`, 添加以下内容:

```yaml
# mermaid chart
mermaid: ## mermaid url https://github.com/knsv/mermaid
  enable: true  # default true
  version: "7.1.2" # default v7.1.2
  options:  # find more api options from https://github.com/knsv/mermaid/blob/master/src/mermaidAPI.js
    #startOnload: true  // default true
```

版本和参数可以根据 [mermaid GitHub](https://github.com/mermaid-js/mermaid) 进行配置

使用默认主题可以直接修改 `themes/landscape/layout/_partial/after-footer.ejs` 添加

```
<% if (theme.mermaid.enable){ %>
  <script src='https://unpkg.com/mermaid@<%= theme.mermaid.version %>/dist/mermaid.min.js'></script>
  <script>
    if (window.mermaid) {
      // mermaid.initialize({theme: 'forest'});
      mermaid.initialize();
    }
  </script>
<% } %>
```

使用 next 主题修改 `themes/next/layout/_partials/footer.swig`, 添加

```
{%- if theme.mermaid.enable %}
  <script src='https://unpkg.com/mermaid@{{ theme.mermaid.version }}/dist/mermaid.min.js'></script>
  <script>
    if (window.mermaid) {
      mermaid.initialize({{ JSON.stringify(theme.mermaid.options) }});
    }
  </script>
{%- endif %}
```