---
title: Markdown 特殊字符
date: 2020-05-04 11:00:00
tags: 'Markdown'
categories:
  - ['开发', 'Markdown']
permalink: markdown-special-characters
---

## 反引号

如何标注一个带反引号的行内 code 片段呢, 最简单的方法就是使用更多的反引号 ``aa`bb``, 这里采用了一下写法

```
``aa`bb``
```

## 反斜杆转义

Markdown 使用 `\` 来忽略或者转义以下特殊字符

| 转义字符 | 英文名称 | 中文名称 |
| -- | -- | -- |
| `\` | backslash | 反斜杠 |
| `` ` `` | backtick | 反引号 |
| `*` | asterisk | 星号 |
| `_` | underscore | 下划线 |
| `{}` | curly braces | 大括号 |
| `[]` | square brackets | 方括号 |
| `()` | parentheses | 括弧 |
| `#` | hash mark | 井号 |
| `+` | plus sign | 加号 |
| `-` | minus sign (hyphen) | 减号（连字符） |
| `.` | dot | 小数点 |
| `!` | exclamation mark | 感叹号 |

## ASCII 转义

常用 html 转义字符

| 字符 | 十进制 | 转义字符 |
| -- | -- | -- |
| `"` | `&#34;` | `&quot;` |
| `&` | `&#38;` | `&amp;` |
| `<` | `&#60;` | `&lt;` |
| `>` | `&#62;` | `&gt;` |
| 不断开空格 (non-breaking space) | `&#160;` | `&nbsp;` |

## 通过 html 标签使用特殊字符

<em>*literal asterisks*</em>
