---
title: match-web-url
date: 2021-03-06 16:00:00
tags: 'Regexp'
categories:
  - ['开发', 'Regexp']
permalink: match-web-url
---

https://blog.walterlv.com/post/match-web-url-using-regex.html

# 使用正则表达式尽可能准确匹配域名 / 网址

2019-12-09 08:58

你可能需要准确地知道一段字符串是否是域名 / 网址 /URL. 虽然可以使用 `.`,`/` 这些来模糊匹配, 但会造成误判.

实际上单纯使用正则表达式来精确匹配也是非常复杂的, 通过代码来判断会简单很多. 不过本文依然从域名的定义出发来尽可能匹配一段字符串是否是域名或者网址, 在要求不怎么高的场合, 使用本文的正则表达式写的代码会比较简单.

* * *

## 网址

网址实际上是 URL(统一资源定位符), 它是由协议, 主机名和路径组成. 不过我们通常所说的网址中的主机名通常是域名, 因此我们在匹配的时候主要考虑域名.

### 域名

[维基百科](https://zh.wikipedia.org/wiki/%E5%9F%9F%E5%90%8D) 中关于域名的描述:

> 1.  域名由一或多个部分组成, 这些部分通常连接在一起, 并由点分隔. 最右边的一个标签是顶级域名, 例如 zh.wikipedia.org 的顶级域名是 org. 一个域名的层次结构, 从右侧到左侧隔一个点依次下降一层. 每个标签可以包含 1 到 63 个八字节. 域名的结尾有时候还有一点, 这是保留给根节点的, 书写时通常省略, 在查询时由软件内部补上.
> 2.  域名里的英文字母不区分大小写.
> 3.  完整域名的所有字符加起来不得超过 253 个 ASCII 字符的总长度. 因此, 当每一级都使用单个字符时, 限制为 127 个级别:127 个字符加上 126 个点的总长度为 253. 但实际上, 某些域名可能具有其他限制; 也没有只有一个字符的域名后缀.

后面关于非 ASCII 字符的描述我没有贴出来. 这种域名例如 ". 中国 ".

在 [中国电信网站备案自助管理系统](http://beian.ct10000.com/portal/icp/help/helpdetail.do? helpid=305) 中, 我们可以找到关于域名的描述:

> 域名中的标号都由英文字母和数字组成, 每一个标号不超过 63 个字符, 也不区分大小写字母. 标号中除连字符 (-) 外不能使用其他的标点符号. 级别最低的域名写在最左边, 而级别最高的域名写在最右边. 由多个标号组成的完整域名总共不超过 255 个字符.

### 路径

路径是使用 `/` 分隔的一段一段字符串.

## 正则表达式匹配

在确认了完整的网址 URL 的规范之后, 使用正则表达式来匹配就会比较精确了.

### 域名

现在, 我们来尝试匹配一下域名 .

1.  每个标签可组成的字符是 `-` `a-z` `A-Z` `0-9`, 但是 `-` 不可作为开头, 标签总长度 1-63 个字符, 于是
    *   `[a-zA-Z0-9][-a-zA-Z0-9]{0,62}`
    *   即首字不含 `-`, 后面的字可以包含 `-`
2.  允许多个标签, 于是
    *   `(\.[a-zA-Z0-9][-a-zA-Z0-9]{0,62})+`
    *   除了标签内容和前面一样, 但我们加了个 `.`

别忘了, 我们还有总长度限制, 于是考虑加上零宽断言 `^.{3,255}$`, 匹配开头和结尾, 中间任意字符但长度在 3-255 之间. 通过零宽断言, 我们可以在不捕获匹配字符串的情况下对后面的字符串增加限制条件.

现在, 把整个正则表达式拼出来:

```
^(?=^.{3,255}$) [a-zA-Z0-9][-a-zA-Z0-9]{0,62} (\.[a-zA-Z0-9][-a-zA-Z0-9]{0,62})+$
```

### URL

对于不同的业务需求, 可能有严格匹配或者宽松的匹配方式.

比如你要做一些比较精准的检查时需要进行严格的检查, 那么选择严格匹配; 这时, 稍微出现一些不符合要求的字符都将认定为不是 URL.

如果你只是打算做一些简单的检查 (例如只是语法高亮), 那么简单匹配即可; 因为当你使用 Chrome 浏览器访问这些 URL 的时候, 依然可以正常访问, Chrome 会帮你格式化一下这个 URL.

*   [https://blog.walterlv.com/post/read-32bit-registry-from-x64-process.html](https://blog.walterlv.com/post/read-32bit-registry-from-x64-process.html)
    *   严格匹配和宽松匹配都会成功匹配
*   [https://blog.lindexi.com/post/dotnet- 配置 -github- 自动打包上传 -nuget- 文件.html](https://blog.lindexi.com/post/dotnet- 配置 -github- 自动打包上传 -nuget- 文件.html)
    *   里面有 Unicode 字符, 宽松匹配才可以匹配此 URL
    *   你把这个 URL 复制到 Chrome 中可以正常打开, 但从 Chrome 里把这个复制出来的话, 就会被转义成 [https://blog.lindexi.com/post/dotnet-%E9%85%8D%E7%BD%AE-github-%E8%87%AA%E5%8A%A8%E6%89%93%E5%8C%85%E4%B8%8A%E4%BC%A0-nuget-%E6%96%87%E4%BB%B6.html](https://blog.lindexi.com/post/dotnet-%E9%85%8D%E7%BD%AE-github-%E8%87%AA%E5%8A%A8%E6%89%93%E5%8C%85%E4%B8%8A%E4%BC%A0-nuget-%E6%96%87%E4%BB%B6.html).
*   [https://\[2001:4860:4860::8888\]:53/favicon.svg](https://[2001:4860:4860::8888]:53/favicon.svg)
    *   因为我偷懒了, 所以只有宽松匹配才可以匹配此 IPv6 地址下的 URL
*   [https:// 域名. 中国](https:// 域名. 中国)
    *   因为我偷懒了, 所以只有宽松匹配才可以匹配此 IPv6 地址下的 URL

#### URL(严格)

匹配 URL 跟匹配域名不一样, URL 复杂得多. 严格匹配的要求是准确反应出 URL 的标准, 但实际上如实反应标准编写的正则表达式会非常复杂, 因此相比于 100% 准确匹配, 我们还是从简了.

所以如果不是有特别要求, 建议还是跳到后面的 " 宽松 " 部分来阅读吧!

我们以下面这个网址为例说明.

[https://blog.walterlv.com/post/read-32bit-registry-from-x64-process.html](https://blog.walterlv.com/post/read-32bit-registry-from-x64-process.html)

1.  前面是可选的协议名, 于是
    *   `(http(s)?:\/\/)`
    *   然而既然可选, 而且是行首, 那么加一个 `?` 和什么都不加的效果是一样的
2.  随后是域名, 于是
    *   `[a-zA-Z0-9][-a-zA-Z0-9]{0,62} (\.[a-zA-Z0-9][-a-zA-Z0-9]{0,62})+`
    *   这里我们没有把总长度限制算上去
3.  别忘了有个可选的端口号
    *   `(: [0-9]{1,5})?`
    *   端口号的范围是 0-65535, 但 0 是保留端口,49152 到 65535 也是保留端口, 因此可以作为网址访问的范围也就是 1-49151, 因此我们限制 1-5 位长度.
4.  接下来是资源路径
    *   资源路径可以使用的字符也是有限制的, 我们接下来详细说明.

组合整个正则表达式:

```
^[a-zA-Z0-9][-a-zA-Z0-9]{0,62} (\.[a-zA-Z0-9][-a-zA-Z0-9]{0,62})+ (: [0-9]{1,5})? [-a-zA-Z0-9()@:%_\\\+\.~#?&//=]*$
```

顺便一提, 不同于域名, 我们这里去掉了长度限制, 因为 URL 真的可以 " 很长 ". 另外, 这里的

现在, 我们补充说明一下资源路径可以使用的字符问题.

`;` `/` `?` `:` `@` `&` `=` `+` `$` `,` 这些字符应该被转义. 转义使用的字符是 `&xxx;`, 因此在转义之后, 依然还可能在网址中看到 `&` 和 `;`, 不过没有其他字符了.

`-` `_` `.` `!` `~` `*` `'` `(` `)` 这些字符可以不进行转义, 但也不建议在 URL 中使用. 对于这部分, 我们考虑将其匹配.

`{` `}` `|` `\` `^` `[` `]` `` ` `` 这部分字符可能被网关当作分隔符使用, 因此不建议出现在 URL 中. 对于这部分, 我们考虑将其匹配.

`<` `>` `#` `%` `"` 控制字符. 使用 `%` 可以组成其他 Unicode 字符, 使用 `#` 用来指代网址中的某个部分.

因此, 我们最终总结应该匹配的特殊字符有 `@` `:` `%` `_` `\` `+` `.` `~` `#` `?` `&` `/` `=`.

#### URL(宽松)

宽松一点的话, 正则表达式就好写多了.

这个正则表达式可以不写 `https` 协议前缀:

```
^\w+ [^\s]+ (\.[^\s]+) {1,}$
```

如果上下文中要求必须匹配 `https`, 则可以写:

```
^(http(s)?:\/\/)\w+ [^\s]+ (\.[^\s]+) {1,}$
```

*   `https://blog.walterlv.com/post/read-32bit-registry-from-x64-process.html#content)`
    *   期望不匹配 (主要是不能匹配末尾的括号), 实际匹配
    *   在 URL 中, 如果括号是成对的, 则此 URL 允许以 `)` 结尾, 如果 URL 中括号不成对, 则此 URL 不能以 `)` 结尾;`>` 同理
*   `https://blog.walterlv.com/post/read-32bit -registry-from-x64-process.html`
    *   期望不匹配, 实际不匹配
*   `https://blog.lindexi.com/post/dotnet- 配置 -github- 自动打包上传 -nuget- 文件.html`
    *   期望匹配, 实际匹配
*   `https:// 域名. 中国 `
    *   期望匹配, 实际匹配
*   `blog.walterlv.com/post/read-32bit-registry-from-x64-process.html`
    *   期望匹配, 实际匹配
*   `x<blog.walterlv.com/post/read-32bit-registry-from-x64-process.html`
    *   期望不匹配, 实际匹配

这里的宽松正则表达式请小心! 此正则表达式会将一段话中 URL 后面非空格的部分都算作 URL 的一部分.

## 更多大牛匹配 URL 的正则表达式

在 GitHub 上还有很多大牛们在写各种匹配 URL 的正则表达式:

*   [regex-weburl.js](https://gist.github.com/dperini/729294)

最长的一个写了 1347 个字符, 最短的有 38 个字符.

有人将其整理成一张表格, 一图说明各种正则表达式能匹配到什么程度:

*   [In search of the perfect URL validation regex](https://mathiasbynens.be/demo/url-regex)

* * *

**参考资料**

*   [In search of the perfect URL validation regex](https://mathiasbynens.be/demo/url-regex)
*   [域名 - 维基百科, 自由的百科全书](https://zh.wikipedia.org/wiki/%E5%9F%9F%E5%90%8D)
*   [中国电信网站备案自助管理系统](http://beian.ct10000.com/portal/icp/help/helpdetail.do? helpid=305)

本文会经常更新, 请阅读原文: [https://blog.walterlv.com/post/match-web-url-using-regex.html](/post/match-web-url-using-regex.html) , 以避免陈旧错误知识的误导, 同时有更好的阅读体验.

如果你想持续阅读我的最新博客, 请点击 [RSS 订阅](/feed.xml), 或者 [前往 CSDN 关注我的主页](https://walterlv.blog.csdn.net/).

 [! [知识共享许可协议](https://blog.walterlv.com/assets/img/by-nc-sa.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/) 本作品采用 [知识共享署名 - 非商业性使用 - 相同方式共享 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-sa/4.0/) 进行许可. 欢迎转载, 使用, 重新发布, 但务必保留文章署名 吕毅 (包含链接: [https://blog.walterlv.com](https://blog.walterlv.com) ), 不得用于商业目的, 基于本文修改后的作品务必以相同的许可发布. 如有任何疑问, 请 [与我联系 (\[email protected\])](/cdn-cgi/l/email-protection#582f39342c3d2a76342e182929763b3735) .