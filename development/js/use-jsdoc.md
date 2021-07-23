---
title: use-jsdoc
date: 2021-03-06 17:00:00
tags: 'JavaScript'
categories:
  - ['开发', 'JavaScript']
permalink: use-jsdoc
---

https://www.jianshu.com/p/d7819c519a47

# jsdoc 的使用

最近要为公司项目做 javaScript API, 在网上找了几个生成工具, JSDoc 和 YUIDoc 好像很不错. 我选用 JSDoc 制作, 用 node 部署后使用起来很方便. 由于是刚开始学, 所以很多地方弄不清楚, 比如 jsdoc 的命名空间 / 路径, 还有对 es6 的支持, getter/setter 怎么写等等. 希望了解的同学帮忙解答~~恩谢

* * *

## 安装

#### jsdoc

确保安装了 node 后, 输入命令    

> npm i jsdoc  -g 

#### **IDE 插件**

sublime 安装插件  **DocBlockr**

vscode  安装 ** add jsdoc comments**

方便自动生成注释

此外, 安装 JSDoc 的**ES6**支持插件  [jsdoc-export-default-interop](https://link.jianshu.com? t= https://www.npmjs.com/package/jsdoc-export-default-interop)

> $ npm i  jsdoc-export-default-interop --save-dev

#### docstrap

由于 JSDoc 默认的文档模板比较单调, 而 docstrap 提供了 14+ 种 bootstrap 风格的模板, 因此建议下载 docstrap

npm i ink-docstrap

或者访问 [github 项目地址](https://link.jianshu.com? t= https://github.com/docstrap/docstrap)

下载完成, 进入到 docstrap 目录安装一下依赖包 npm install

移除 google 字体, 防止页面卡顿:

打开 docstrap\\template\\static\\styles , 将引用的 google 字体内容删除

> @import url("https://fonts.googleapis.com/css? family= Roboto:400,500");

指定模板: 在 jsdoc 的配置文件 conf.json 下的 template 选项 配置为 docstrap/template 即可 

如果要手动修改模板样式: 修改文件 docstrap\\template\\tmpl\\details.tmpl

* * *

## 使用

1. 新建配置文件**conf.json**

为了养成良好的使用习惯, 提升工作效率, 建议一开始就配置好配置文件. 配置文件的具体格式和参数详见官方文档

2. 拷贝你的 js 到你配置文件的指定目录

3. 输入命令

> jsdoc  c  conf.json  

会生成一个 out 文件夹, 里面是 jsdoc 生成的 API.

你还可以导入项目的 README.md, package.json, 附件, 教程等等.

参考资料:

[JSDoc 中文指南](https://link.jianshu.com? t= http://www.css88.com/doc/jsdoc/index.html)

[csdn 网友博客](https://link.jianshu.com? t= http://blog.csdn.net/wts/article/details/19255357)

附上我 test 用的 conf.json 文件截图

此外, tutorials 里的 json 目录配置文件大概如下

对 ES6 的支持:

1. 安装 [jsdoc-export-default-interop](https://link.jianshu.com? t= https://www.npmjs.com/package/jsdoc-export-default-interop) 插件

命令行安装, 并在 conf.json 里添加接口

> npm install jsdoc-export-default-interop --save-dev

> "plugins":\["../node\_modules/jsdoc-export-default-interop/dist/index"\]

2\. 将 exports default class xx 改为 class  xx .. module.exports =  xx ... 或者 class xx ... export default xx .. 

贴上几个常用的块级标签:

@argument 指定参数名和说明来描述一个函数参数.

@return

@example 函数使用示例

@returns 描述函数的返回值.

@author 指示代码的作者.

@deprecated 指示一个函数已经废弃, 不建议使用, 而且在将来版本的代码中可能会彻底删除. 要避免使用这段代码.

@see 创建一个 HTML 链接指向指定类的描述.

@version 指定发布版本.

@requires 创建一个 HTML 链接, 指向这个类所需的指定类.

@throws

@exception 描述函数可能抛出的异常的类型.

@link 创建一个 HTML 链接, 指向指定的类. 这与@see 很类似, 但@link 能嵌在注释文本中. @author 指示代码的作者.(译者注: 这个标记前面已经出现过, 建议去掉)

@fileoverview 这是一个特殊的标记, 如果在文件的第一个文档块中使用这个标记, 则指定该文档块的余下部分将用来提供文件的一个概述.

@class 提供类的有关信息, 用在构造函数的文档中.

@constructor 明确一个函数是某个类的构造函数.

@type 指定函数的返回类型.

@extends 指示一个类派生了另一个类. 通常 JSDoc 自己就可以检测出这种信息, 不过, 在某些情况下则必须使用这个标记.

@private 指示一个类或函数是私有的. 私有类和函数不会出现在 HTML 文档中, 除非运行 JSDoc 时提供了 ---private 命令行选项.

@final 指示一个值是常量值. 要记住 JavaScript 无法真正保证一个值是常量.

@ignore JSDoc 会忽略有这个标记的函数.

### 推荐阅读 [更多精彩内容](/)

*   [Spring Cloud](/p/46fd0faecac1)

    Spring Cloud 为开发人员提供了快速构建分布式系统中一些常见模式的工具 (例如配置管理, 服务发现, 断路器, 智...

    [卡卡罗 2017](/u/d90908cb0d85) 阅读 111,950 评论 15 赞 132

    [](/p/46fd0faecac1)
*   [Spring boot 参考指南](/p/67a0e41dfe05)

    Spring Boot 参考指南 介绍 转载自: https://www.gitbook.com/book/qbgb...

    [毛宇鹏](/u/d3ea915e1e0f) 阅读 41,021 评论 6 赞 343

*   [Android - 收藏集](/p/dad51f6c9c4d)

    Android 自定义 View 的各种姿势 1 Activity 的显示之 ViewRootImpl 详解 Activity...

    [passiontim](/u/e946d18f163c) 阅读 152,214 评论 22 赞 666

*   [webpack 教程](/p/5b69a7e61fe4)

    无意中看到 zhangwnag 大佬分享的 webpack 教程感觉受益匪浅, 特此分享以备自己日后查看, 也希望更多的人看到...

    [小小字符](/u/1689862fc5a0) 阅读 6,633 评论 6 赞 33

    [](/p/5b69a7e61fe4)
*   [感恩日志 第 [796] 天: (2017.11.12)](/p/0876b2bc5fdb)

    财富似水在流动, 水聚集在地处, 有源头, 有去处. 财富是有限的, 不在这里就在那里, 不用在事上, 就会用在人上. 不是马...

    [慧恩咨询贺守仁](/u/53bd094591c9) 阅读 68 评论 1 赞 0
