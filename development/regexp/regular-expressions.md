---
title: 正则表达式
date: 2020-05-05 8:00:00
tags: 'Regexp'
categories:
  - ['开发', 'Regexp']
permalink: regular-expressions
mathjax: true
---

## 简介

正则表达式可以用形式化语言理论的方式来表达, 正则表达式由常量和算子组成, 它们分别表示字符串的集合和在这些集合上的运算, 给定有限字母表Σ定义了下列常量:

- 空集 ${\displaystyle \varnothing }$ 表示集合 ${\displaystyle \varnothing }$
- 空串 ${\displaystyle \varepsilon }$ 表示仅包含一个 "不含任何字符, 长度为 0 的字符串" 的集合
- 文字字符 ${\displaystyle a\in \Sigma }$ 表示仅包含一个元素 ${\displaystyle a}$ 的集合 ${\displaystyle \{a\}}$

定义了下列运算:

<escape>
- 串接 ${\displaystyle RS}$ 表示集合 ${\displaystyle \{\alpha \beta \mid \alpha \in R,\beta \in S\}}$, 这里的 ${\displaystyle \alpha \beta }$ 表示将 ${\displaystyle \alpha }$ 和 ${\displaystyle \beta }$ 两个字符串按顺序连接, 例如: ${\displaystyle \{ab,c\}\{d,ef\}=\{abd,abef,cd,cef\}}$
- 选择 ${\displaystyle R|S}$ 表示 ${\displaystyle R}$ 和 ${\displaystyle S}$ 的并集, 例如: ${\displaystyle \{ab,c\}|\{ab,d,ef\}=\{ab,c,d,ef\}}$
- 克莱尼 (Kleene) 星号 ${\displaystyle R^{*}}$ 表示包含 ${\displaystyle \varepsilon }$ 且在字符串串接运算下闭合的 ${\displaystyle R}$ 的最小超集, 这是可以通过 ${\displaystyle R}$ 中零或有限个字符串的串接得到所有字符串的集合, 例如: ${\displaystyle \{ab,c\}^{*}=\{\varepsilon ,ab,c,abab,abc,cab,cc,ababab,\cdots \}}$
- 上述常量和算子形成了克莱尼代数
</escape>

很多课本使用对选择使用符号 ${\displaystyle \cup }$, ${\displaystyle +}$ 或 ${\displaystyle \vee }$ 替代竖线

为了避免括号, 假定 Kleene 星号有最高优先级, 接着是串接, 接着是并集, 如果没有歧义则可以省略括号, 例如: `(ab)c` 可以写为 `abc` 而 `a|(b(c*))` 可以写为 `a|bc*`

例子:

- `a|b*` 表示 ${\displaystyle \{\varepsilon ,a,b,bb,bbb,\cdots \}}$
- `(a|b)*` 表示包括空串和任意数目个 `a` 或 `b` 字符组成的所有字符串的集合: ${\displaystyle \{\varepsilon ,a,b,aa,ab,ba,bb,aaa\cdots \}}$
- `ab*(c|ε)` 表示开始于一个 `a` 接着零或多个 `b` 和最后一个可选的 c 组成的字符串的集合: ${\displaystyle \{a,ac,ab,abc,abb,abbc\cdots \}}$
- 为了使表达式更简洁, 正则表达式也定义了 `?` 和 `+`, `aa*` 等于 `a+`, 表示 `a` 出现至少一次, 而 `(a|ε)` 等于 `a?`, 表示 `a` 出现 `1` 次或不出现, 有的定义中增加了补算子 ${\displaystyle \sim }$, ${\displaystyle \sim R}$ 表示在 ${\displaystyle \Sigma ^{*}}$ 上但不在 ${\displaystyle R}$ 中的所有字符串的集合, 补算子在理论上并非必要, 因为它可以使用其他算子来表达, 但它可以使一些表达式变得更加简洁

这种意义上的正则表达式可以表达正则语言, 是可被有限状态自动机精确接受的语言类, 但是在简洁性上有重要区别, 某类正则语言只能用大小指数增长的自动机来描述, 而要求的正则表达式的长度只线性的增长

正则表达式对应于乔姆斯基层级的类型-3 文法, 但通常编程语言或其相关库 (例如 PCRE) 中实现的正则表达式的表达能力是乔姆斯基层级中类型-3 文法的超集 [ [来源请求](https://zh.wikipedia.org/wiki/Wikipedia:%E5%88%97%E6%98%8E%E6%9D%A5%E6%BA%90) ], 在另一方面, 在正则表达式和不导致这种大小上的爆炸的非确定有限状态自动机 (NFA) 之间有简单的映射, 为此 NFA 经常被用作正则表达式的替表示式

这种形式化中存在着冗余, 典型的体现是存在不同的正则表达式可以表达同样的语言, 有可能对两个给定正则表达式写一个算法来判定它们所描述的语言是否本质上相等, 即简约每个表达式到极小确定有限自动机, 确定它们是否同构 (等价) , 这种冗余可以消减到什么程度？我们可以找到仍有完全表达力的正则表达式的有趣的子集吗？这提出了一个令人惊奇的困难问题, Kleene 星号和并集明显是需要的, 但是我们或许可以限制它们的使用, 由于正则表达式如此简单, 没有办法在语法上把它重写成某种规范形式, 过去公理化的缺乏导致了星号高度问题, 最近 Dexter Kozen 用克莱尼代数公理化了正则表达式, [ [来源请求](https://zh.wikipedia.org/wiki/Wikipedia:%E5%88%97%E6%98%8E%E6%9D%A5%E6%BA%90) ]

很多现实世界的 "正则表达式" 引擎实现了不能用正则表达式代数表达的特征, [ [来源请求](https://zh.wikipedia.org/wiki/Wikipedia:%E5%88%97%E6%98%8E%E6%9D%A5%E6%BA%90) ]

<!-- more -->

## 基本语法

一个正则表达式通常被称为一个模式 (pattern) , 为用来描述或者匹配一系列符合某个句法规则的字符串, 例如: Handel, Händel 和 Haendel 这三个字符串, 都可以由 `H(a|ä|ae)ndel` 这个模式来描述, 大部分正则表达式的形式都有如下的结构:

### 选择

竖线 `|` 代表选择 (即或集) , 具有最低优先级, 例如 `gray|grey` 可以匹配 grey 或 gray

### 数量限定

某个字符后的数量限定符用来限定前面这个字符允许出现的个数, 最常见的数量限定符包括`+`, `?`和 `*` (不加数量限定则代表出现一次且仅出现一次) :

- 加号 `+` 代表前面的字符必须至少出现一次,  (`1` 次或多次) , 例如, `goo+gle` 可以匹配 `google`, `gooogle`, `goooogle` 等
- 问号 `?` 代表前面的字符最多只可以出现一次,  (`0` 次或 `1` 次) , 例如, `colou?r` 可以匹配 `color` 或者 `colour`
- 星号 `*` 代表前面的字符可以不出现, 也可以出现一次或者多次,  (`0` 次, `1` 次或多次) , 例如, `0*42` 可以匹配 `42`, `042`, `0042`, `00042` 等

### 匹配

圆括号 `()` 可以用来定义操作符的范围和优先度, 例如, `gr(a|e)y` 等价于 `gray|grey`, `(grand)?father` 匹配 `father` 和 `grandfather`

上述这些构造子都可以自由组合, 因此 `H(ae?|ä)ndel` 和 `H(a|ae|ä)ndel` 是相同的, 表示 `{"Handel", "Haendel", "Händel"}`

精确的语法可能因不同的工具或程序而异

## PCRE 表达式全集

正则表达式有多种不同的风格, 下表是在 PCRE 中元字符及其在正则表达式上下文中的行为的一个完整列表, 适用于 Perl 或者 Python 编程语言 (grep 或者 egrep 的正则表达式文法是 PCRE 的子集)

| `字符` | 描述 |
| -- | -- |
| `\` | 将下一个字符标记为一个特殊字符 (File Format Escape, 清单见本表) , 或一个原义字符 (Identity Escape, 有 `^$()*+?.[\{|` 共计 `12` 个), 或一个向后引用 (backreferences) , 或一个八进制转义符, 例如,  `n` 匹配字符 `n` ,  `\n` 匹配一个换行符, 序列 `\\` 匹配 `\` 而 `\(` 则匹配 `(`  |
| `^` | 匹配输入字符串的开始位置, 如果设置了 RegExp 对象的 Multiline 属性, `^` 也匹配 `\n` 或 `\r` 之后的位置 |
| `$` | 匹配输入字符串的结束位置, 如果设置了 RegExp 对象的 Multiline 属性, `$` 也匹配 `\n` 或 `\r` 之前的位置 |
| `*` | 匹配前面的子表达式零次或多次, 例如, `zo*` 能匹配 `z` ,  `zo` 以及 `zoo` , `*` 等价于 `{0,}` |
| `+` | 匹配前面的子表达式一次或多次, 例如,  `zo+` 能匹配 `zo` 以及 `zoo` , 但不能匹配 `z` , `+` 等价于 `{1,}` |
| `?` | 匹配前面的子表达式零次或一次, 例如,  `do(es)?` 可以匹配 `does` 中的 `do` 和 `does` , `?` 等价于 `{0,1}` |
| `{n}` | `n` 是一个非负整数, 匹配确定的 `n` 次, 例如,  `o{2}` 不能匹配 `Bob` 中的 `o` , 但是能匹配 `food` 中的两个 `o` |
| `{n,}` | `n` 是一个非负整数, 至少匹配 `n` 次, 例如,  `o{2,}` 不能匹配 `Bob` 中的 `o` , 但能匹配 `foooood` 中的所有 `o`,  `o{1,}` 等价于 `o+` ,  `o{0,}` 则等价于 `o*`  |
| `{n,m}` | `m` 和 `n` 均为非负整数, 其中 `n<=m`, 最少匹配 `n` 次且最多匹配 `m` 次, 例如,  `o{1,3}` 将匹配 `fooooood` 中的前三个 `o`,  `o{0,1}` 等价于 `o?` , 请注意在逗号和两个数之间不能有空格 |
| `?` | 非贪心量化 (Non-greedy quantifiers) : 当该字符紧跟在任何一个其他重复修饰符 `(*,+,?, {n}, {n,}, {n,m})` 后面时, 匹配模式是非贪婪的, 非贪婪模式尽可能少的匹配所搜索的字符串, 而默认的贪婪模式则尽可能多的匹配所搜索的字符串, 例如, 对于字符串 `oooo` ,  `o+?` 将匹配单个 `o` , 而 `o+` 将匹配所有 `o`  |
| `.` | 匹配除 `\r`  `\n` 之外的任何单个字符, 要匹配包括 `\r`  `\n` 在内的任何字符, 请使用像 `(.|\r|\n)` 的模式 |
| `(pattern)` | 匹配 pattern 并获取这一匹配的子字符串, 该子字符串用于向后引用, 所获取的匹配可以从产生的 Matches 集合得到, 在 VBScript 中使用 SubMatches 集合, 在 JScript 中则使用 `$0…$9` 属性, 要匹配圆括号字符, 请使用 `\(` 或 `\)` , 可带数量后缀 |
| `(?:pattern)` | 匹配 pattern 但不获取匹配的子字符串 (shy groups) , 也就是说这是一个非获取匹配, 不存储匹配的子字符串用于向后引用, 这在使用或字符 `(|)` 来组合一个模式的各个部分是很有用, 例如 `industr(?:y|ies)` 就是一个比 `industry|industries` 更简略的表达式 |
| `(?=pattern)` | 正向肯定预查 (look ahead positive assert) , 在任何匹配 pattern 的字符串开始处匹配查找字符串, 这是一个非获取匹配, 也就是说, 该匹配不需要获取供以后使用, 例如,  `Windows(?=95|98|NT|2000)` 能匹配 `Windows2000` 中的 `Windows` , 但不能匹配 `Windows3.1` 中的 `Windows` , 预查不消耗字符, 也就是说, 在一个匹配发生后, 在最后一次匹配之后立即开始下一次匹配的搜索, 而不是从包含预查的字符之后开始 |
| `(?!pattern)` | 正向否定预查 (negative assert) , 在任何不匹配 pattern 的字符串开始处匹配查找字符串, 这是一个非获取匹配, 也就是说, 该匹配不需要获取供以后使用, 例如 `Windows(?!95|98|NT|2000)` 能匹配 `Windows3.1` 中的 `Windows` , 但不能匹配 `Windows2000` 中的 `Windows` , 预查不消耗字符, 也就是说, 在一个匹配发生后, 在最后一次匹配之后立即开始下一次匹配的搜索, 而不是从包含预查的字符之后开始 |
| `(?<=pattern)` | 反向 (look behind) 肯定预查, 与正向肯定预查类似, 只是方向相反, 例如,  `(?<=95|98|NT|2000)Windows` 能匹配 `2000Windows` 中的 `Windows` , 但不能匹配 `3.1Windows` 中的 `Windows`  |
| `(?<!pattern)` | 反向否定预查, 与正向否定预查类似, 只是方向相反, 例如 `(?<!95|98|NT|2000)Windows` 能匹配 `3.1Windows` 中的 `Windows` , 但不能匹配 `2000Windows` 中的 `Windows`  |
| `x|y` | 没有包围在 `()` 里, 其范围是整个正则表达式, 例如,  `z|food` 能匹配 `z` 或 `food` ,  `(?:z|f)ood` 则匹配 `zood` 或 `food`  |
| `[xyz]` | 字符集合 (character class) , 匹配所包含的任意一个字符, 例如,  `[abc]` 可以匹配 `plain` 中的 `a` , 特殊字符仅有反斜线 `\` 保持特殊含义, 用于转义字符, 其它特殊字符如星号, 加号, 各种括号等均作为普通字符, 脱字符 `^` 如果出现在首位则表示负值字符集合, 如果出现在字符串中间就仅作为普通字符, 连字符 `-` 如果出现在字符串中间表示字符范围描述, 如果如果出现在首位 (或末尾) 则仅作为普通字符, 右方括号应转义出现, 也可以作为首位字符出现 |
| `[^xyz]` | 排除型字符集合 (negated character classes) , 匹配未列出的任意字符, 例如,  `[^abc]` 可以匹配 `plain` 中的 `plin`  |
| `[a-z]` | 字符范围, 匹配指定范围内的任意字符, 例如,  `[a-z]` 可以匹配 `a` 到 `z` 范围内的任意小写字母字符 |
| `[^a-z]` | 排除型的字符范围, 匹配任何不在指定范围内的任意字符, 例如,  `[^a-z]` 可以匹配任何不在 `a` 到 `z` 范围内的任意字符 |
| `[:name:]` | 增加命名字符类 (named character class) [ 注 1] 中的字符到表达式, 只能用于方括号表达式 |
| `[=elt=]` | 增加当前 locale 下排序 (collate) 等价于字符 `elt` 的元素, 例如, `[=a=]` 可能会增加 `ä, á, à, ă, ắ, ằ, ẵ, ẳ, â, ấ, ầ, ẫ, ẩ, ǎ, å, ǻ, ä, ǟ, ã, ȧ, ǡ, ą, ā, ả, ȁ, ȃ, ạ, ặ, ậ, ḁ, ⱥ, ᶏ, ɐ, ɑ` , 只能用于方括号表达式 |
| `[.elt.]` | 增加排序元素 (collation element) elt 到表达式中, 这是因为某些排序元素由多个字符组成, 例如, `29` 个字母表的西班牙语,  `CH`作为单个字母排在字母 `C` 之后, 因此会产生如此排序 `cinco, credo, chispa` , 只能用于方括号表达式 |
| `\b` | 匹配一个单词边界, 也就是指单词和空格间的位置, 例如,  `er\b` 可以匹配 `never` 中的 `er` , 但不能匹配 `verb` 中的 `er`  |
| `\B` | 匹配非单词边界,  `er\B` 能匹配 `verb` 中的 `er` , 但不能匹配 `never` 中的 `er`  |
| `\cx` | 匹配由 `x` 指明的控制字符, `x` 的值必须为 `A-Z` 或 `a-z` 之一, 否则, 将 `c` 视为一个原义的 `c` 字符, 控制字符的值等于 `x` 的值最低 `5` 比特 (即对 `3210` 进制的余数) , 例如, `\cM` 匹配一个 Control-M 或回车符, `\ca` 等效于 `\u0001`, `\cb` 等效于 `\u0002`, 等等 … |
| `\d` | 匹配一个数字字符, 等价于 `[0-9]`, 注意 Unicode 正则表达式会匹配全角数字字符 |
| `\D` | 匹配一个非数字字符, 等价于 `[^0-9]` |
| `\f` | 匹配一个换页符, 等价于 `\x0c` 和 `\cL` |
| `\n` | 匹配一个换行符, 等价于 `\x0a` 和 `\cJ` |
| `\r` | 匹配一个回车符, 等价于 `\x0d` 和 `\cM` |
| `\s` | 匹配任何空白字符, 包括空格, 制表符, 换页符等等, 等价于 `[ \f\n\r\t\v]`, 注意 Unicode 正则表达式会匹配全角空格符 |
| `\S` | 匹配任何非空白字符, 等价于 `[^ \f\n\r\t\v]` |
| `\t` | 匹配一个制表符, 等价于 `\x09` 和 `\cI` |
| `\v` | 匹配一个垂直制表符, 等价于 `\x0b` 和 `\cK` |
| `\w` | 匹配包括下划线的任何单词字符, 等价于 `[A-Za-z0-9_]` , 注意 Unicode 正则表达式会匹配中文字符 |
| `\W` | 匹配任何非单词字符, 等价于 `[^A-Za-z0-9_]`  |
| `\xnn` | 十六进制转义字符序列, 匹配两个十六进制数字 nn 表示的字符, 例如,  `\x41` 匹配 `A` ,  `\x041` 则等价于 `\x04&1` , 正则表达式中可以使用 ASCII 编码, . |
| `\num` | 向后引用 (back-reference) 一个子字符串 (substring) , 该子字符串与正则表达式的第 `num` 个用括号围起来的捕捉群 (capture group) 子表达式 (subexpression) 匹配, 其中 `num` 是从 `1` 开始的十进制正整数, 其上限可能是 `9`, `31`, `99` 甚至无限, 例如:  `(.)\1` 匹配两个连续的相同字符 |
| `\n` | 标识一个八进制转义值或一个向后引用, 如果 `\n` 之前至少 `n` 个获取的子表达式, 则 `n` 为向后引用, 否则, 如果 `n` 为八进制数字 `(0-7)` , 则 `n` 为一个八进制转义值 |
| `\nm` | `3` 位八进制数字, 标识一个八进制转义值或一个向后引用, 如果 `\nm` 之前至少有 `nm` 个获得子表达式, 则 `nm` 为向后引用, 如果 `\nm` 之前至少有 `n` 个获取, 则 `n` 为一个后跟文字 `m` 的向后引用, 如果前面的条件都不满足, 若 `n` 和 `m` 均为八进制数字 `(0-7)` , 则 `\nm` 将匹配八进制转义值 `nm` |
| `\nml` | 如果 `n` 为八进制数字 `(0-3)` , 且 `m` 和 `l` 均为八进制数字 `(0-7)` , 则匹配八进制转义值 `nml` |
| `\un` | Unicode 转义字符序列, 其中 `n` 是一个用四个十六进制数字表示的 Unicode 字符, 例如, `\u00A9` 匹配著作权符号 `(©)`  |

## Unicode 处理

在.NET, Java, JavaScript, Python 的正则表达式中, 可以用 `\uXXXX` 表示一个 Unicode 字符, 其中 XXXX 为四位 `16` 进制数字

Unicode 字符的三种性质:

- Unicode Property: 字符属于标点, 空格, 字母等等, 每个 Unicode 字符只能属于唯一 Unicode Property, .NET, Java, PHP 和 Ruby 等语言支持, 具体分类为:
  - 字符 `\p{L}`
    - `\p{Ll}` 或`\p{Lowercase_Letter}`: 小写字符 (必须有大写的形式)
    - `\p{Lu}` 或`\p{Uppercase_Letter}`: 大写字符 (必须有小写的形式)
    - `\p{Lt}` 或`\p{Titlecase_Letter}`: 全词首字母大写的字符
    - `\p{L&}` 或`\p{Cased_Letter}`: 存在大小写形式的字符 (`Ll`, `Lu`, `Lt` 的组合)
    - `\p{Lm}` 或`\p{Modifier_Letter}`: 音标修饰字符
    - `\p{Lo}` 或`\p{Other_Letter}`: 不具有大小写的字符或字形
  - 附加符号`\p{M}`
    - `\p{Mn}` 或`\p{Non_Spacing_Mark}`: 与其他字符结合, 不额外占用空间的字符, 例如日耳曼语元音变音
    - `\p{Mc}` 或`\p{Spacing_Combining_Mark}`: 与其他字符结合, 额外占用空间的字符, 例如马拉雅拉姆文#元音字母及附标
    - `\p{Me}` 或`\p{Enclosing_Mark}`: 包含其他字符的字符, 例如圆圈, 方块
  - 分隔符 `p{Z}`
    - `\p{Zs}` 或`\p{Space_Separator}`: a whitespace character that is invisible, but does take up space
    - `\p{Zl}` 或`\p{Line_Separator}`: line separator character `U+2028`
    - `\p{Zp}` 或`\p{Paragraph_Separator}`: paragraph separator character `U+2029`
  - 符号`\p{S}`
    - `\p{Sm}` 或`\p{Math_Symbol}`: 数学符号
    - `\p{Sc}` 或`\p{Currency_Symbol}`: 通货符号
    - `\p{Sk}` 或`\p{Modifier_Symbol}`: 组合为其他字符的符号
    - `\p{So}` 或`\p{Other_Symbol}`: 其他符号
  - 数值字符`\p{N}`
    - `\p{Nd}` 或`\p{Decimal_Digit_Number}`: 所有文本中的数字 `0` 至 `9` 字符, 不含形意符号
    - `\p{Nl}` 或`\p{Letter_Number}`: 看起来像字母的符号, 包含罗马数字
    - `\p{No}` 或`\p{Other_Number}`: 上角标或下角标数字, 或者其他不属于 `0` 至 `9` 的数字, 不含形意符号
  - 标点符号`\p{P}`
    - `\p{Pd}` 或`\p{Dash_Punctuation}`: 任何种类的连字号或连接号
    - `\p{Ps}` 或`\p{Open_Punctuation}`: 任何种类开括号
    - `\p{Pe}` 或`\p{Close_Punctuation}`: 任何种类闭括号
    - `\p{Pi}` 或`\p{Initial_Punctuation}`: 任何种类开引号
    - `\p{Pf}` 或`\p{Final_Punctuation}`: 任何种类闭引号
    - `\p{Pc}` 或`\p{Connector_Punctuation}`: 连接词的标点符号, 如下划线
    - `\p{Po}` 或`\p{Other_Punctuation}`: 其他标点符号
  - 其它符号`\p{C}` (包括不可见控制字符与未用码位)
    - `\p{Cc}` 或`\p{Control}`: ASCII 或 Latin-1 控制字符 `0x00-0x1F` 与 `0x7F-0x9F`
    - `\p{Cf}` 或`\p{Format}`: 不可见的格式化指示字符
    - `\p{Co}` 或`\p{Private_Use}`: 私用码位
    - `\p{Cs}` 或`\p{Surrogate}`: `UTF-16` 编码的代理对的一半
    - `\p{Cn}` 或`\p{Unassigned}`: 未被使用的码位
- Unicode Block: 按照编码区间划分 Unicode 字符, 每个 Unicode Block 中的字符编码属于一个编码区间, 例如 Java 语言`\p{ InCJK_Compatibility_Ideographs }`, .NET 语言`\p{IsCJK_Compatibility_Ideographs}`
- Unicode Script: 按照字符所属的书写系统来划分 Unicode 字符, PHP 和 Ruby (版本不低于 1.9) 支持 Unicode Script, 例如`\p{Han}` 表示汉字 (中文字符)

这三种 Unicode 性质对应的字符组补集是将开头的`\p` 改为`\P`, 其它不变

## POSIX 字符组

| `POSIX` 字符组 | 说明 | ASCII 环境 | Unicode 环境 |
| `--` | -- | -- | -- |
| `[:alnum:]` | 字母字符和数字字符 | `[a-zA-Z0-9]` | `[\p{L&}\p{Nd}]` |
| `[:alpha:]` | 字母 | `[a-zA-Z]` | `\p{L&}` |
| `[:ascii:]` | ASCII 字符 | `[\x00-\x7F]` | `\p{InBasicLatin}` |
| `[:blank:]` | 空格字符和制表符 | `[ \t]` | `[\p{Zs}\t]` |
| `[:cntrl:]` | 控制字符 | `[\x00-\x1F\x7F]` | `\p{Cc}` |
| `[:digit:]` | 数字字符 | `[0-9]` | `\p{Nd}` |
| `[:graph:]` | 空白字符之外的字符 | `[\x21-\x7E]` | `[^\p{Z}\p{C}]` |
| `[:lower:]` | 小写字母字符 | `[a-z]` | `\p{Ll}` |
| `[:print:]` | 类似 `[:graph:]`, 但包括空白字符 | `[\x20-\x7E]` | `\P{C}` |
| `[:punct:]` | 标点符号 | ``[][!"#$%&'()*+,./:;<=>?@\^_`{|}~-]`` | `[\p{P}\p{S}]` |
| `[:space:]` | 空白字符 | `[ \t\r\n\v\f]` | `[\p{Z}\t\r\n\v\f]` |
| `[:upper:]` | 大写字母字符 | `[A-Z]` | `\p{Lu}` |
| `[:word:]` | 字母字符 | `[A-Za-z0-9_]` | `[\p{L}\p{N}\p{Pc}]` |
| `[:xdigit:]` | 十六进制字符 | `[A-Fa-f0-9]` | `[A-Fa-f0-9]` |

## 优先权

最高: `\`
高: `()`, `(?:)`, `(?=)`, `[]`
中: `*`, `+`, `?`, `{n}`, `{n,}`, `{n,m}`
低: `^`, `$`, 中介字符
次最低: 串接, 即相邻字符连接在一起
最低: `|`
