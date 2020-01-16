---
title: Git Commit 规范
date: 2020-1-16 19:00:00
tags: 'Git'
categories:
  - ['使用说明', '软件']
permalink: git-commit-standard-guide
photo:
---

# 简介

Git Commit message 是提交代码前的说明, 主要提供当前变化的目的, 所以应该清晰明了.

[Angular 规范](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit) 是目前使用最广的写法, 比较合理和系统化, 并且有配套的工具.

<!-- more -->

# 作用

## 提供命令行详细的信息

```sh
notes git:(master) ✗ git log --oneline --decorate --graph

* 46dda7c docs: 添加目录
* 6714dd4 Initial commit
```

## 方便命令行过滤条件

```sh
notes git:(master) ✗ git log --oneline --decorate --graph --grep docs

* 5db4688 docs(markdown): 添加 markdown 使用 html 文档
* 208b8d9 docs(live): 添加解决插入图片问题的说明文档
```

## 直接从 commit 生成 Change log

Change Log 是发布新版本时, 用来说明与上一个版本差异的文档

## 定位代码显示修改历史

```sh
notes git:(master) ✗ git log -L 2,3:README.md

commit 799f591d343195bdc63c689088c0869372270b6e
Author: yanjinzh6 <yanjinzh6@gmail.com>
Date:   Sun Dec 22 00:04:22 2019 +0800

    docs: 移动旧文档

diff --git a/README.md b/README.md
--- a/README.md
+++ b/README.md
@@ -1,1 +2,2 @@
-# notes
+title: 博客目录
+date: 2019-12-21 17:15:54

commit 6714dd461612839156123753ad2895fd6707c037
Author: YanjinZhu <yanjinzh6@gmail.com>
Date:   Thu Apr 11 14:56:13 2019 +0800

    Initial commit
```

总的来说, 对个人和团队还有项目的规范有毕竟大的提高

# 格式

每次提交, Commit message 都包括三个部分: `header`, `body` 和 `footer`.

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

其中, `header` 是必需的, `body` 和 `footer` 可以省略.
不管是哪一个部分, 任何一行都不得超过 72 个字符 (或 100 个字符) . 这是为了避免自动换行影响美观.

## Header

`Header` 部分只有一行, 包括三个字段: `type` (必需) , `scope` (可选) 和 `subject` (必需) .

### type

用于说明 commit 的类别, 只允许使用下面 7 个标识.

- feat: 新功能 (feature)
- fix: 修补 bug
- docs: 文档 (documentation)
- style:  格式 (不影响代码运行的变动)
- refactor: 重构 (即不是新增功能, 也不是修改 bug 的代码变动)
- test: 增加测试
- chore: 构建过程或辅助工具的变动

如果 `type` 为 feat 和 fix, 则该 commit 将肯定出现在 Change log 之中. 其他情况 (`docs`, `chore`, `style`, `refactor`, `test`) 由你决定, 要不要放入 Change log.

### scope

`scope` 用于说明 commit 影响的范围, 比如数据层, 控制层, 视图层等等, 视项目不同而不同.

例如在 Angular, 可以是 `$location`, `$browser`, `$compile`, `$rootScope`, `ngHref`, `ngClick`, `ngView` 等.

如果你的修改影响了不止一个 `scope`, 你可以使用 * 代替.

### subject

`subject` 是 commit 目的的简短描述, 不超过 50 个字符.

其他注意事项:

- 以动词开头, 使用第一人称现在时, 比如 `change`, 而不是 `changed` 或 `changes`
- 第一个字母小写
- 结尾不加句号 (.)

## Body

`Body` 部分是对本次 commit 的详细描述, 可以分成多行. 下面是一个范例.

```
More detailed explanatory text, if necessary.  Wrap it to
about 72 characters or so.

Further paragraphs come after blank lines.

- Bullet points are okay, too
- Use a hanging indent
```

有两个注意点:

- 使用第一人称现在时, 比如使用 `change` 而不是 `changed` 或 `changes`.
- 永远别忘了第 2 行是空行
- 应该说明代码变动的动机, 以及与以前行为的对比.

## Footer

`Footer` 部分只用于以下两种情况:

### 不兼容变动

如果当前代码与上一个版本不兼容, 则 `Footer` 部分以 `BREAKING CHANGE` 开头, 后面是对变动的描述, 以及变动理由和迁移方法.

```
BREAKING CHANGE: isolate scope bindings definition has changed.

    To migrate the code follow the example below:

    Before:

    scope: {
      myAttr: 'attribute',
    }

    After:

    scope: {
      myAttr: '@',
    }

    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

### 关闭 Issue

如果当前 commit 针对某个 issue, 那么可以在 `Footer` 部分关闭这个 issue .

```
Closes #234
```

## Revert

还有一种特殊情况, 如果当前 commit 用于撤销以前的 commit, 则必须以 revert:开头, 后面跟着被撤销 Commit 的 Header.

```
revert: feat(pencil): add 'graphiteWidth' option

This reverts commit 667ecc1654a317a13331b17617d973392f415f02.
```

`Body` 部分的格式是固定的, 必须写成 `This reverts commit <hash>.`, 其中的 hash 是被撤销 commit 的 SHA 标识符.

如果当前 commit 与被撤销的 commit, 在同一个发布 (release) 里面, 那么它们都不会出现在 Change log 里面. 如果两者在不同的发布, 那么当前 commit, 会出现在 Change log 的 `Reverts` 小标题下面.

# 工具

## Commitizen

[Commitizen](https://github.com/commitizen/cz-cli) 是一款命令行添加提交消息格式的插件

## validate-commit-msg

[validate-commit-msg](
https://github.com/kentcdodds/validate-commit-msg) 用于检查项目的 Commit message 是否符合 Angular 规范.

## 生成 Change log

如果你的所有 Commit 都符合 Angular 格式, 那么发布新版本时,  Change log 就可以用脚本自动生成. 生成的文档包括以下三个部分:

- New features
- Bug fixes
- Breaking changes.

每个部分都会罗列相关的 commit, 并且有指向这些 commit 的链接. 当然, 生成的文档允许手动修改, 所以发布前, 你还可以添加其他内容.

[conventional-changelog](https://github.com/ajoslin/conventional-changelog) 就是生成 Change log 的工具.
