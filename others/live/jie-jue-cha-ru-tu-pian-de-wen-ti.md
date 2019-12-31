---
title: 解决插入图片的问题
date: 2019-12-23 9:54:00
tags: '博客'
categories:
  - ['其他', '生活']
permalink: jie-jue-cha-ru-tu-pian-de-wen-ti
---

# 起因

手动从 GitHub 中迁移过来就碰到点小问题.

* 每个 md 文件都需要手动插入一块 hexo 的头部模版
* 生成的 h1, h2, h3 的标题确实不太好看, 还有一堆重复的字眼
* 文件中存在的图片混乱
  * 由于之前觉得少量的图片应该比较统一管理, 所以就使用了多个子类型公用一个 images 文件夹的模式, 但是 hexo 需要将所有的图片都放置在 source/images 中, 这样会导致图片失去了版本管理, 还需要同步维护图片
  * 使用 hexo 都资源文件模式, 就得使用 hexo 推荐的脚本语法, 这样造成 GitHub 或者编辑器无法解析

<!-- more -->

# 了解

## hexo 官方推荐

通过修改配置文件开启文章资源文件夹的功能

```yml
# _config.yml
post_asset_folder: true
```

通过在 markdown 中使用相对路径引用标签插件

```md
{% asset_path slug %}
{% asset_img slug [title] %}
{% asset_link slug [title] %}

eg:
![这样使用会出现的问题是无法在首页显示](/example.jpg)

正确的引用图片方式是使用下列的标签插件
{% asset_img example.jpg This is an example image %}
```

## hexo asset image 插件

网络上提示使用 [hexo-asset-image]([https://link](https://www.npmjs.com/package/hexo-asset-image)) 插件

确保开启文章资源文件夹功能, 然后就可以通过正常的 markdown 语法插入图片, 生成过程会自动处理首页图片的问题

然而看到更新时间是 2018 年的, 粗略看了下实现, 也没对编辑器本地预览这种做处理

## 测试

经过测试, hexo 的生成方式是这样的, 结构如下的目录

```
source/_posts
├── example-post
| └── first.jpg
└── example-post.md
```

在 markdown 文件中, 会使用 `![first](example-post/first.jpg)` 来进行显示, 这时候 GitHub 和本地编辑器是可以进行预览的

使用 `npx hexo g` 会生成如下结构

```
public
|
└─ example-post
  |
  └─ first.jpg
  |
  └─ index.html
|
└─ index.html
```

这样在首页和 `example-post/index.html` 中都会渲染成 `<img src="example-post/first.jpg" alt="first">`, 很明显首页是能正常显示的, 但在文章中就不行

所以解决的方式就是重写一个插件, 将文章里面渲染的方式修改为使用当前文件夹下面的图片即可

# 解决

## 生命在于折腾

1. 确定需求, 生成插件目录, 必须是 hexo- 开头的
2. 实现渲染方式正确渲染图片
3. 确定 package.json 中 name, version, main 属性
4. 可以发布到官方 [插件列表](https://hexo.io/plugins) 或者 npm 中
5. 详情参考 [插件](https://hexo.io/zh-cn/docs/plugins)
6. ...

## 而我...

只能通过简单无脑的修改生成后的 html 来实现, 由于采用脚本来实现同步到静态站点, 刚好也可以一并处理

```js

/**
 * 通过对比文件夹名称来修改 html 中图片路径
 * 逻辑就是递归整个目录, 当目录中存在包含 html 文件和资源文件的子目录时,
 * 就判断 html 中是否包含目录名称的图片路径
 * @param {String} fileName 文件全路径
 * @param {String} parentFolder 文件夹名称
 */
function replaceImageUrl(fileName, parentFolder) {
  if (!parentFolder) {
    return
  }
  ext = path.extname(fileName)
  if (ext !== '.html') {
    return
  }
  console.log('当前文件夹为: ', parentFolder)
  const data = fs.readFileSync(fileName, { encoding: 'UTF-8' })
  if (!data) {
    return
  }
  const cur = `<img src="${parentFolder}/`
  const reg = new RegExp(cur, 'g')
  if (data.indexOf(cur) < 0) {
    return
  }
  console.log('匹配文件: ', fileName, cur)
  const result = data.replace(reg, '<img src="')
  fs.writeFileSync(fileName, result, { encoding: 'UTF-8' })
  console.log('写入文件: ', fileName)
}
```

对于几个文件夹和文本处理确实方便也挺快的, 使用同步方法主要是避免多异步操作

# 验证

只是单纯手工检查一下.
