---
title: markdown-basic
date: 2021-03-07 11:00:00
tags: 'Markdown'
categories:
  - ['开发', 'Markdown']
permalink: markdown-basic
---

https://www.markdown.xyz/basic-syntax/#headings

      Markdown 基本语法 | Markdown 指南中文版

 [! [](/assets/images/markdown-mark-white.png) Markdown 指南中文版](/)

[入门指南](/getting-started/) [基本语法](/basic-syntax/) [扩展语法](/extended-syntax/) [速查表](/cheat-sheet/) [书写工具](/tools/) [相关书籍](/book/)

# Markdown 基本语法

John Gruber 的 Markdown 设计文档中列出的所有 Markdown 元素.

## 总览

几乎所有 Markdown 应用程序都支持 John Gruber 原始设计文档中列出的 Markdown 基本语法. 但是, Markdown 处理程序之间存在着细微的变化和差异, 我们都会尽可能标记出来.

## 标题 (Headings)

要创建标题, 请在单词或短语前面添加井号 (`#`) . 井号的数量代表了标题的级别. 例如, 添加三个井号即创建一个三级标题 (`<h3>`) (例如:`### My Header`).

| Markdown | HTML | 渲染效果 |
| --- | --- | --- |
| `# Heading level 1` | `<h1> Heading level 1</h1>` |
# Heading level 1

 |
| `## Heading level 2` | `<h2> Heading level 2</h2>` |

## Heading level 2

 |
| `### Heading level 3` | `<h3> Heading level 3</h3>` |

### Heading level 3

 |
| `#### Heading level 4` | `<h4> Heading level 4</h4>` |

#### Heading level 4

 |
| `##### Heading level 5` | `<h5> Heading level 5</h5>` |

##### Heading level 5

 |
| `###### Heading level 6` | `<h6> Heading level 6</h6>` |

###### Heading level 6

 |

### 可选语法

还可以在文本下方添加任意数量的 `==` 号来标识一级标题, 或者 `--` 号来标识二级标题.

| Markdown | HTML | 渲染效果 |
| --- | --- | --- |
| `Heading level 1
===============` | `<h1> Heading level 1</h1>` |
# Heading level 1

 |
| `Heading level 2
---------------` | `<h2> Heading level 2</h2>` |

## Heading level 2

 |

### 标题 (Heading) 用法的最佳实践

当井号 (`#`) 和标题文本之间没有空格时, 各 Markdown 应用程序的处理方式是不一样的. 为了兼容考虑, 请在井号和标题文本之间添加一个空格.

| ✅  这样做 | ❌  不要这样做 |
| --- | --- |
| `# Here's a Heading

` | `#Here's a Heading` |

## 段落 (Paragraphs)

要创建段落, 请使用空白行将一行或多行文本进行分隔.

| Markdown | HTML | 渲染效果 |
| --- | --- | --- |
| `I really like using Markdown.

I think I'll use it to format all of my documents from now on.` | `<p> I really like using Markdown.</p>

<p> I think I'll use it to format all of my documents from now on.</p>` |
I really like using Markdown.

I think I'll use it to format all of my documents from now on.

 |

### 段落 (Paragraph) 用法的最佳实践

除非 [段落在列表中](/basic-syntax/#paragraphs), 否则不要用空格 (spaces) 或制表符 ( tabs) 缩进段落.

| ✅  这样做 | ❌  不要这样做 |
| --- | --- |
| `Don't put tabs or spaces in front of your paragraphs.

Keep lines left-aligned like this.

` |  `This can result in unexpected formatting problems.

  Don't add tabs or spaces in front of paragraphs.` |

## 换行 (Line Breaks)

在一行的末尾添加两个或多个空格, 然后按回车键 (return), 即可创建一个换行 (line break) (`<br>`).

| Markdown | HTML | 渲染效果 |
| --- | --- | --- |
| `This is the first line.  
And this is the second line.` | `<p> This is the first line.<br>
And this is the second line.</p>` |
This is the first line.
And this is the second line.

 |

### 换行 (Line Break) 用法的最佳实践

几乎每个 Markdown 应用程序都支持两个或多个空格进行换行 (称为 " 结尾空格 (trailing whitespace)") 的方式, 但这是有争议的, 因为很难在编辑器中直接看到空格, 并且很多人在每个句子后面都会有意或无意地添加两个空格. 由于这个原因, 你可能需要使用除结尾空格以外的其它方式来进行换行. 幸运的是, 几乎每个 Markdown 应用程序都支持另一种换行方式: HTML 的 `<br>` 标签.

为了兼容性, 请在行尾添加 " 结尾空格 " 或 HTML 的 `<br>` 标签来实现换行.

还有两种其他方式我并不推荐使用.CommonMark 和其它几种轻量级标记语言支持在行尾添加反斜杠 (`\`) 的方式实现换行, 但是并非所有 Markdown 应用程序都支持此种方式, 因此从兼容性的角度来看, 不推荐使用. 并且至少有两种轻量级标记语言支持无须在行尾添加任何内容, 只须键入回车键 ( return) 即可实现换行.

| ✅  这样做 | ❌  不要这样做 |
| --- | --- |
| `First line with two spaces after.  
And the next line.

First line with the HTML tag after.<br>
And the next line.

` | `First line with a backslash after.\
And the next line.

First line with nothing after.
And the next line.

` |

## 强调 (Emphasis)

通过将文本设置为粗体或斜体来强调其重要性.

### 粗体 (Bold)

要加粗文本, 请在单词或短语的前后各添加两个星号 (asterisks) 或下划线 (underscores). 如需加粗一个单词或短语的中间部分用以表示强调的话, 请在要加粗部分的两侧各添加两个星号 (asterisks).

| Markdown | HTML | 渲染效果 |
| --- | --- | --- |
| `I just love **bold text**.` | `I just love <strong> bold text</strong>.` | I just love **bold text**. |
| `I just love __bold text__.` | `I just love <strong> bold text</strong>.` | I just love **bold text**. |
| `Love**is**bold` | `Love<strong> is</strong> bold` | Love**is**bold |

#### 粗体 (Bold) 用法最佳实践

Markdown 应用程序在如何处理单词或短语中间的下划线上并不一致. 为兼容考虑, 在单词或短语中间部分加粗的话, 请使用星号 (asterisks).

| ✅  这样做 | ❌  不要这样做 |
| --- | --- |
| `Love**is**bold` | `Love__is__bold` |

### 斜体 (Italic)

要用斜体显示文本, 请在单词或短语前后添加一个星号 (asterisk) 或下划线 (underscore). 要斜体突出单词的中间部分, 请在字母前后各添加一个星号, 中间不要带空格.

| Markdown | HTML | 渲染效果 |
| --- | --- | --- |
| `Italicized text is the *cat's meow*.` | `Italicized text is the <em> cat's meow</em>.` | Italicized text is the _cat's meow_. |
| `Italicized text is the _cat's meow_.` | `Italicized text is the <em> cat's meow</em>.` | Italicized text is the _cat's meow_. |
| `A*cat*meow` | `A<em> cat</em> meow` | A_cat_meow |

#### 斜体 (Italic) 用法的最佳实践

Markdown 的众多应用程序在如何处理单词中间的下划线上意见不一致. 为了兼容起见, 请用星号标注单词中间的斜体来表示着重.

| ✅  这样做 | ❌  不要这样做 |
| --- | --- |
| `A*cat*meow` | `A_cat_meow` |

### 粗体 (Bold) 和斜体 (Italic)

要同时用粗体和斜体突出显示文本, 请在单词或短语的前后各添加三个星号或下划线. 要加粗并用斜体显示单词或短语的中间部分, 请在要突出显示的部分前后各添加三个星号, 中间不要带空格.

| Markdown | HTML | 渲染效果 |
| --- | --- | --- |
| `This text is ***really important***.` | `This text is <strong> <em> really important</em> </strong>.` | This text is **_really important_**. |
| `This text is ___really important___.` | `This text is <strong> <em> really important</em> </strong>.` | This text is **_really important_**. |
| `This text is __*really important*__.` | `This text is <strong> <em> really important</em> </strong>.` | This text is **_really important_**. |
| `This text is **_really important_**.` | `This text is <strong> <em> really important</em> </strong>.` | This text is **_really important_**. |
| `This is really***very***important text.` | `This is really<strong> <em> very</em> </strong> important text.` | This is really**_very_**important text. |

#### 粗体 (Bold) 和斜体 (Italic) 用法的最佳实践

Markdown 应用程序在处理单词或短语中间添加的下划线上并不一致. 为了实现兼容性, 请使用星号将单词或短语的中间部分加粗并以斜体显示, 以示重要.

| ✅  这样做 | ❌  不要这样做 |
| --- | --- |
| `This is really***very***important text.` | `This is really___very___important text.` |

## 块引用 (Blockquotes)

要创建块引用, 请在段落前添加一个 `>` 符号.

```
> Dorothy followed her through many of the beautiful rooms in her castle.
```

渲染效果如下所示:

> Dorothy followed her through many of the beautiful rooms in her castle.

### 多个段落的块引用 (Blockquotes)

块引用可以包含多个段落. 为段落之间的空白行各添加一个 `>` 符号.

```
> Dorothy followed her through many of the beautiful rooms in her castle.
>
> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.
```

渲染效果如下:

> Dorothy followed her through many of the beautiful rooms in her castle.
>
> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.

### 嵌套块引用 (Nested Blockquotes)

块引用可以嵌套. 在要嵌套的段落前添加一个 `>>` 符号.

```
> Dorothy followed her through many of the beautiful rooms in her castle.
>
>> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.
```

渲染效果如下:

> Dorothy followed her through many of the beautiful rooms in her castle.
>
> > The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.

### 带有其它元素的块引用 (Blockquotes with Other Elements)

块引用可以包含其他 Markdown 格式的元素. 并非所有元素都可以使用, 你需要进行实验以查看哪些元素有效.

```
> #### The quarterly results look great!
>
> - Revenue was off the chart.
> - Profits were higher than ever.
>
>  *Everything* is going according to **plan**.
```

渲染效果如下:

> #### The quarterly results look great!
>
> *   Revenue was off the chart.
> *   Profits were higher than ever.
>
> _Everything_ is going according to **plan**.

## 列表 (Lists)

你可以将多个条目组织成有序或无序列表.

### 有序列表 (Ordered Lists)

要创建有序列表, 请在每个列表项前添加数字并紧跟一个英文句点. 数字不必按数学顺序排列, 但是列表应当以数字 1 起始.

| Markdown | HTML | Rendered Output |
| --- | --- | --- |
| `1. First item
2. Second item
3. Third item
4. Fourth item` | `<ol>
<li> First item</li>
<li> Second item</li>
<li> Third item</li>
<li> Fourth item</li>
</ol>` |
1.  First item
2.  Second item
3.  Third item
4.  Fourth item

 |
| `1. First item
1. Second item
1. Third item
1. Fourth item` | `<ol>
<li> First item</li>
<li> Second item</li>
<li> Third item</li>
<li> Fourth item</li>
</ol>` |

1.  First item
2.  Second item
3.  Third item
4.  Fourth item

 |
| `1. First item
8. Second item
3. Third item
5. Fourth item` | `<ol>
<li> First item</li>
<li> Second item</li>
<li> Third item</li>
<li> Fourth item</li>
</ol>` |

1.  First item
2.  Second item
3.  Third item
4.  Fourth item

 |
| `1. First item
2. Second item
3. Third item
    1. Indented item
    2. Indented item
4. Fourth item` | `<ol>
<li> First item</li>
<li> Second item</li>
<li> Third item
<ol>
<li> Indented item</li>
<li> Indented item</li>
</ol>
</li>
<li> Fourth item</li>
</ol>` |

1.  First item
2.  Second item
3.  Third item
    1.  Indented item
    2.  Indented item
4.  Fourth item

 |

#### 有序列表 (Ordered List) 用法的最佳实践

CommonMark 和其它几种轻量级标记语言可以让你使用括号 (`)`) 作为分隔符 (例如 `1) First item`), 但并非所有的 Markdown 应用程序都支持此种用法, 因此, 从兼容的角度来看, 此用法不推荐. 为了兼容起见, 请只使用英文句点作为分隔符.

| ✅  这样做 | ❌  不要这样做 |
| --- | --- |
| `1. First item
2. Second item` | `1) First item
2) Second item` |

### 无序列表 (Unordered Lists)

要创建无序列表, 请在每个列表项前面添加破折号 (`-`), 星号 (`*`) 或加号 (`+`) . 缩进一个或多个列表项可创建嵌套列表.

| Markdown | HTML | 渲染效果 |
| --- | --- | --- |
| `- First item
- Second item
- Third item
- Fourth item` | `<ul>
<li> First item</li>
<li> Second item</li>
<li> Third item</li>
<li> Fourth item</li>
</ul>` |
*   First item
*   Second item
*   Third item
*   Fourth item

 |
| `* First item
* Second item
* Third item
* Fourth item` | `<ul>
<li> First item</li>
<li> Second item</li>
<li> Third item</li>
<li> Fourth item</li>
</ul>` |

*   First item
*   Second item
*   Third item
*   Fourth item

 |
| `+ First item
+ Second item
+ Third item
+ Fourth item` | `<ul>
<li> First item</li>
<li> Second item</li>
<li> Third item</li>
<li> Fourth item</li>
</ul>` |

*   First item
*   Second item
*   Third item
*   Fourth item

 |
| `- First item
- Second item
- Third item
    - Indented item
    - Indented item
- Fourth item` | `<ul>
<li> First item</li>
<li> Second item</li>
<li> Third item
<ul>
<li> Indented item</li>
<li> Indented item</li>
</ul>
</li>
<li> Fourth item</li>
</ul>` |

*   First item
*   Second item
*   Third item
    *   Indented item
    *   Indented item
*   Fourth item

 |

#### 无序列表 (Unordered List) 用法的最佳实践

Markdown 应用程序在如何处理同一列表中混用不同分隔符上并不一致. 为了兼容起见, 请不要在同一个列表中混用不同的分隔符, 最好选定一种分隔符并一直用下去.

| ✅  这样做 | ❌  不要这样做 |
| --- | --- |
| `- First item
- Second item
- Third item
- Fourth item` | `+ First item
* Second item
- Third item
+ Fourth item` |

### 在列表中添加列表项

要在保留列表连续性的同时在列表中添加另一种元素, 请将该元素缩进四个空格或一个制表符, 如下例所示:

#### 段落 (Paragraphs)

```
*   This is the first list item.
*   Here's the second list item.

    I need to add another paragraph below the second list item.

*   And here's the third list item.
```

渲染效果如下:

*   This is the first list item.
*   Here's the second list item.

    I need to add another paragraph below the second list item.

*   And here's the third list item.

#### 引用块 (Blockquotes)

```
*   This is the first list item.
*   Here's the second list item.

    > A blockquote would look great below the second list item.

*   And here's the third list item.
```

渲染效果如下:

*   This is the first list item.
*   Here's the second list item.

    > A blockquote would look great below the second list item.

*   And here's the third list item.

#### 代码块 (Code Blocks)

[代码块 (Code blocks)](#code-blocks) 通常采用四个空格或一个制表符缩进. 当它们被放在列表中时, 请将它们缩进八个空格或两个制表符.

```
1.  Open the file.
2.  Find the following code block on line 21:

        <html>
          <head>
            <title> Test</title>
          </head>

3.  Update the title to match the name of your website.
```

渲染效果如下:

1.  Open the file.
2.  Find the following code block on line 21:

    ```
    <html>
      <head>
        <title> Test</title>
      </head>
    ```

3.  Update the title to match the name of your website.

#### 图片 (Images)

```
1.  Open the file containing the Linux mascot.
2.  Marvel at its beauty.

    ! [Tux, the Linux mascot](/assets/images/tux.png)

3.  Close the file.
```

渲染效果如下:

1.  Open the file containing the Linux mascot.
2.  Marvel at its beauty.

    ! [Tux, the Linux mascot](/assets/images/tux.png)

3.  Close the file.

#### 列表 (Lists)

你可以将无序列表嵌套在有序列表中, 反之亦然.

```
1. First item
2. Second item
3. Third item
    - Indented item
    - Indented item
4. Fourth item
```

渲染效果如下:

1.  First item
2.  Second item
3.  Third item
    *   Indented item
    *   Indented item
4.  Fourth item

## 代码

要将单词或短语表示为代码, 请将其包裹在反引号 (`` ` ``) 中.

| Markdown | HTML | 渲染效果 |
| --- | --- | --- |
| ``At the command prompt, type `nano`.`` | `At the command prompt, type <code> nano</code>.` | At the command prompt, type `nano`. |

### 转义反引号

如果你要表示为代码的单词或短语中包含一个或多个反引号, 则可以通过将单词或短语包裹在双反引号 (` `` `) 中.

| Markdown | HTML | 渲染效果 |
| --- | --- | --- |
| ``` ``Use `code` in your Markdown file.`` ``` | ``<code> Use `code` in your Markdown file.</code>`` | ``Use `code` in your Markdown file.`` |

### 代码块 (Code Blocks)

要创建代码块, 请将代码块的每一行缩进至少四个空格或一个制表符.

```
 <html>
      <head>
      </head>
    </html>
```

渲染效果如下:

```
<html>
  <head>
  </head>
</html>
```

**注意:** 要创建不用缩进的代码块, 请使用 [围栏式代码块 (fenced code blocks)](/extended-syntax/#fenced-code-blocks).

## 分隔线 (Horizontal Rules)

要创建分隔线, 请在单独一行上使用三个或多个星号 (`***`), 破折号 (`---`) 或下划线 (`___`) , 并且不能包含其他内容.

```
***

---

_________________
```

以上三个分隔线的渲染效果看起来都一样:

* * *

### 分隔线 (Horizontal Rule) 用法最佳实践

为了兼容性, 请在分隔线的前后均添加空白行.

| ✅  这样做 | ❌  不要这样做 |
| --- | --- |
| `Try to put a blank line before...

---

...and after a horizontal rule.` | `Without blank lines, this would be a heading.
---
Don't do this!` |

## 链接 (Links)

要创建链接, 请将链接文本括在方括号 (例如 `[Duck Duck Go]`) 中, 后面紧跟着括在圆括号中的 URL(例如 `(https://duckduckgo.com)` ).

```
My favorite search engine is [Duck Duck Go](https://duckduckgo.com).
```

渲染效果如下:

My favorite search engine is [Duck Duck Go](https://duckduckgo.com).

### 添加标题

你可以选择为链接添加标题 (即 title 属性). 当用户将鼠标悬停在链接上时, 将显示一个提示. 要添加标题, 请将其放在 URL 后面.

```
My favorite search engine is [Duck Duck Go](https://duckduckgo.com "The best search engine for privacy").
```

渲染效果如下:

My favorite search engine is [Duck Duck Go](https://duckduckgo.com "The best search engine for privacy").

### 网址和电子邮件地址

要将 URL 或电子邮件地址快速转换为链接, 请将其括在尖括号中.

```
<https://www.markdownguide.org>
<fake@example.com>
```

渲染效果如下:

[https://www.markdownguide.org](https://www.markdownguide.org)
[fake@example.com](mailto: fake@example.com)

### 格式化链接

如需 [强调 (emphasize)](#emphasis) 某个链接, 请在方括号前及圆括号后添加星号. 要将链接表示为 [代码 (code)](#code) , 请在方括号内添加反引号.

```
I love supporting the **[EFF](https://eff.org)**.
This is the *[Markdown Guide](https://www.markdownguide.org)*.
See the section on [`code`](#code).
```

渲染效果如下:

I love supporting the **[EFF](https://eff.org)**.
This is the _[Markdown Guide](https://www.markdownguide.org)_.
See the section on [`code`](#code).

### 引用式链接

引用式 (Reference-style) 链接是一种特殊类型的链接, 它使得 URL 在 Markdown 中更易于显示和阅读. 引用式链接由两部分组成: 一部分被放置在正文文本中; 另一部分被放置在文档中的其它地方, 以便于阅读.

#### 引用式链接第一部分的格式

引用式链接的第一部分的格式由两组方括号组成. 第一组方括号内放的是显示为链接的文本, 第二组方括号内放的是一个标签, 该标签用于指向您存放在文档中其它位置的链接.

尽管不是必须的, 但你可以在第一组和第二组方括号之间添加一个空格. 第二组方括号中的标签不区分大小写, 并且可以包含字母, 数字, 空格或标点符号.

以下示例中, 链接的第一部分是等效的:

*   `[hobbit-hole][1]`
*   `[hobbit-hole] [1]`

#### 引用式链接第二部分的格式

引用式链接的第二部分可以包含以下属性:

1.  放在方括号内的标签, 以及紧跟在方括号后面的一个冒号和至少一个空格 (例如 `[label]:` ).
2.  链接的 URL, 可以选择将其括在尖括号内.
3.  链接的标题 (可有可无), 可以将其括在双引号, 单引号或括号内.

以下示例中, 链接的第二部分是等效的:

*   `[1]: https://en.wikipedia.org/wiki/Hobbit#Lifestyle`
*   `[1]: https://en.wikipedia.org/wiki/Hobbit#Lifestyle "Hobbit lifestyles"`
*   `[1]: https://en.wikipedia.org/wiki/Hobbit#Lifestyle 'Hobbit lifestyles'`
*   `[1]: https://en.wikipedia.org/wiki/Hobbit#Lifestyle (Hobbit lifestyles)`
*   `[1]: <https://en.wikipedia.org/wiki/Hobbit#Lifestyle> "Hobbit lifestyles"`
*   `[1]: <https://en.wikipedia.org/wiki/Hobbit#Lifestyle> 'Hobbit lifestyles'`
*   `[1]: <https://en.wikipedia.org/wiki/Hobbit#Lifestyle> (Hobbit lifestyles)`

可以将链接的第二部分放在 Markdown 文档中的任何位置. 有些人将它们放在被引用的段落的后面, 有些人将它们放在文档的末尾 (类似尾注或脚注).

#### 将两部分组合在一起使用的示例

假设你将一个 URL 作为一个 [标准 URL 链接](#links) 添加到段落中, 在 Markdown 中如下所示:

```
In a hole in the ground there lived a hobbit. Not a nasty, dirty, wet hole, filled with the ends
of worms and an oozy smell, nor yet a dry, bare, sandy hole with nothing in it to sit down on or to
eat: it was a [hobbit-hole](https://en.wikipedia.org/wiki/Hobbit#Lifestyle "Hobbit lifestyles"), and that means comfort.
```

Though it may point to interesting additional information, the URL as displayed really doesn't add much to the existing raw text other than making it harder to read. To fix that, you could format the URL like this instead:

```
In a hole in the ground there lived a hobbit. Not a nasty, dirty, wet hole, filled with the ends
of worms and an oozy smell, nor yet a dry, bare, sandy hole with nothing in it to sit down on or to
eat: it was a [hobbit-hole][1], and that means comfort.

[1]: <https://en.wikipedia.org/wiki/Hobbit#Lifestyle> "Hobbit lifestyles"
```

在上述两个实例中, 渲染后的输出是相同的:

> In a hole in the ground there lived a hobbit. Not a nasty, dirty, wet hole, filled with the ends of worms and an oozy smell, nor yet a dry, bare, sandy hole with nothing in it to sit down on or to eat: it was a [hobbit-hole](https://en.wikipedia.org/wiki/Hobbit#Lifestyle "Hobbit lifestyles"), and that means comfort.

该链接的 HTML 为:

```
<a href="https://en.wikipedia.org/wiki/Hobbit#Lifestyle" title="Hobbit lifestyles"> hobbit-hole</a>
```

### 链接 (Link) 的最佳使用实践

不同的 Markdown 应用程序在处理 URL 中间的空格方面是不一样的. 为了兼容起见, 请尽量使用 `%20` (空格的编码形式) 来代替空格.

| ✅  这样做 | ❌  不要这样做 |
| --- | --- |
| `[link](https://www.example.com/my%20great%20page)` | `[link](https://www.example.com/my great page)` |

## 图片 (Images)

要添加图片, 首先请添加感叹号 (`!`), 然后紧跟着是方括号, 方括号中可添加替代文本 (alt text, 即图片显示失败后显示此文本), 最后跟着圆括号, 圆括号中添加图片资源的路径或 URL. 你可以选择在圆括号中的 URL 之后添加标题 (即 title 属性).

```
! [Philadelphia's Magic Gardens. This place was so cool!](/assets/images/philly-magic-gardens.jpg "Philadelphia's Magic Gardens")
```

渲染效果如下:

! [Philadelphia's Magic Gardens. This place was so cool!](/assets/images/philly-magic-garden.jpg "Philadelphia's Magic Gardens")

### 带链接的图片

要为图片添加链接, 请先为图片的 Markdown 标记添加一个方括号, 然后紧跟着一个圆括号, 并在圆括号中添加链接地址.

```
[! [An old rock in the desert](/assets/images/shiprock.jpg "Shiprock, New Mexico by Beau Rogers")](https://www.flickr.com/photos/beaurogers/31833779864/in/photolist-Qv3rFw-34mt9F-a9Cmfy-5Ha3Zi-9msKdv-o3hgjr-hWpUte-4WMsJ1-KUQ8N-deshUb-vssBD-6CQci6-8AFCiD-zsJWT-nNfsgB-dPDwZJ-bn9JGn-5HtSXY-6CUhAL-a4UTXB-ugPum-KUPSo-fBLNm-6CUmpy-4WMsc9-8a7D3T-83KJev-6CQ2bK-nNusHJ-a78rQH-nw3NvT-7aq2qf-8wwBso-3nNceh-ugSKP-4mh4kh-bbeeqH-a7biME-q3PtTf-brFpgb-cg38zw-bXMZc-nJPELD-f58Lmo-bXMYG-bz8AAi-bxNtNT-bXMYi-bXMY6-bXMYv)
```

渲染效果如下:

[! [An old rock in the desert](/assets/images/shiprock.jpg "Shiprock, New Mexico by Beau Rogers")](https://www.flickr.com/photos/beaurogers/31833779864/in/photolist-Qv3rFw-34mt9F-a9Cmfy-5Ha3Zi-9msKdv-o3hgjr-hWpUte-4WMsJ1-KUQ8N-deshUb-vssBD-6CQci6-8AFCiD-zsJWT-nNfsgB-dPDwZJ-bn9JGn-5HtSXY-6CUhAL-a4UTXB-ugPum-KUPSo-fBLNm-6CUmpy-4WMsc9-8a7D3T-83KJev-6CQ2bK-nNusHJ-a78rQH-nw3NvT-7aq2qf-8wwBso-3nNceh-ugSKP-4mh4kh-bbeeqH-a7biME-q3PtTf-brFpgb-cg38zw-bXMZc-nJPELD-f58Lmo-bXMYG-bz8AAi-bxNtNT-bXMYi-bXMY6-bXMYv)

## 转义字符 (Escaping Characters)

要显示原本用于格式化 Markdown 文档的字符, 请在字符前面添加反斜杠字符 (`\`) .

```
\* 如果没有开头的反斜杠字符的话, 这一行将显示为无序列表.
```

渲染效果如下:

\* 如果没有开头的反斜杠字符的话, 这一行将显示为无序列表.

### 可做转义的 (英文) 字符

以下列出的字符都可以通过使用反斜杠字符从而达到转义目的.

| 字符 | 名称 |
| --- | --- |
| \\ | 反斜杠 (backslash) |
| \` | backtick (另请参见 [在代码中转义反引号](#escaping-backticks)) |
| \* | 星号 (asterisk) |
| \_ | 下划线 (underscore) |
| { } | 花括号 (curly braces) |
| \[ \] | 方括号 (brackets) |
| < > | angle brackets |
| ( ) | 圆括号或括号 (parentheses) |
| # | 井号 (pound sign) |
| + | 加号 (plus sign) |
| \- | 减号 (minus sign) (也叫连字符 hyphen) |
| . | 句点 (dot) |
| ! | 感叹号 (exclamation mark) |
| | | 管道符 (pipe) (另请参见 [在表格中转义管道符](/extended-syntax/#escaping-pipe-characters-in-tables)) |

## HTML 标签

大多 Markdown 应用程序允许你在 Markdown 格式文本中添加 HTML 标签. 如果你喜欢某些 HTML 标签胜于 Markdown 语法的话, 这将何有帮助. 例如, 某些人发现通过 HTML 标签添加图像更加容易. 当你需要更改元素的属性时 (例如为文本指定颜色或更改图像的宽度), 使用 HTML 标签更方便些.

如需使用 HTML, 请将 HTML 标签添加到 Markdown 格式文本中即可.

```
This **word** is bold. This <em> word</em> is italic.
```

渲染效果如下:

This **word** is bold. This _word_ is italic.

### HTML 用法最佳实践

出于安全原因, 并非所有 Markdown 应用程序都支持在 Markdown 文档中添加 HTML. 如有疑问, 请查看 Markdown 应用程序的文档. 某些应用程序只支持 HTML 标签的子集.

对于 HTML 的块级元素 `<div>`,`<table>`,`<pre>` 和 `<p>`, 请在其前后使用空行 (blank lines) 与其它内容进行分隔. 尽量不要使用制表符 (tabs) 或空格 (spaces) 对 HTML 标签做缩进, 否则将影响格式.

在 HTML 块级标签内不能使用 Markdown 语法. 例如 `<p> italic and **bold**</p>` 将不起作用.

[关于](/about/)     [Contact](/contact/)     [GitHub](https://github.com/mattcone/markdown-guide)     [API](/api/v1/)     [Privacy Policy](https://app.termly.io/document/privacy-policy/1ca2b712-96e3-46bf-a8f1-d0035d389e7d)

© 2021. A [Matt Cone](https://www.mattcone.com) project. [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). Made with 🌶️ in [New Mexico](https://www.newmexico.org/).

© 2021 Markdown 指南中文版采用 [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) 许可证发布.

[京 ICP 备 15031610 号 -30](https://beian.miit.gov.cn/)

##### 友链

*   [Bootstrap 中文网](https://www.bootcss.com/)
*   [React](https://reactjs.bootcss.com/)
*   [MDX](https://www.mdxjs.cn/)
*   [Next.js](https://www.nextjs.cn/)
*   [Docusaurus](https://www.docusaurus.cn/)
*   [Gatsby](https://www.gatsbyjs.cn/)
*   [Blitz](https://www.blitzjs.cn/)
*   [Vue](https://vuejs.bootcss.com/)
*   [Nuxt.js](https://www.nuxtjs.cn/)
*   [VuePress](https://www.vuepress.cn/)
*   [Gridsome](https://www.gridsome.cn/)
*   [Jekyll](https://www.jekyll.com.cn/)
*   [Hugo](https://www.gohugo.cn/)
*   [Webpack](https://www.webpackjs.com/)
*   [Rollup.js](https://www.rollupjs.com/)
*   [Parcel](https://www.parceljs.cn/)
*   [WebAssembly](https://www.wasm.com.cn/)
*   [Deno](https://www.denojs.cn/)
