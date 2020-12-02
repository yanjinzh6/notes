---
title: Markdown LaTeX 语法
date: 2020-05-04 11:00:00
tags: 'Markdown'
categories:
  - ['开发', 'Markdown']
permalink: markdown-latex
mathjax: true
---

## 简介

> LATEX 是一种排版系统, 它非常适用于生成高印刷质量的科技和数学类文档, 这个系统同样适用于生成从简单的信件到完整书籍的所有其他种类的文档, LATEX 使用 TEX 作为它的格式化引擎

## 语法

- 行内公式 `$数学公式$`: 公式在文中与文字或其他东西混编
- 独立行公式 `$$数学公式$$`: 公式单独占一行

### 上下标

- `^`: 上标 $n^2$
- `_`: 下标 $T_a$

<!-- more -->

### 分数

使用 `\frac{分子}{分母}`, 推荐使用 `\cfrac` 来代替

`\frac{1}{3} 与 \cfrac{1}{3}`

$$\frac{1}{3} 与 \cfrac{1}{3}$$

### 开方

使用 `\sqrt[次数]{被开方数}`

`\sqrt[3]{X}`

$$\sqrt[3]{X}$$

## 操作符

| Symbol | Command |
| -- | -- |
| $\pm$ | `\pm` |
| $\mp$ | `\mp` |
| $\times$ | `\times` |
| $\div$ | `\div` |
| $\cdot$ | `\cdot` |
| $\ast$ | `\ast` |
| $\star$ | `\star` |
| $\dagger$ | `\dagger` |
| $\ddagger$ | `\ddagger` |
| $\amalg$ | `\amalg` |
| $\cap$ | `\cap` |
| $\cup$ | `\cup` |
| $\uplus$ | `\uplus` |
| $\sqcap$ | `\sqcap` |
| $\sqcup$ | `\sqcup` |
| $\vee$ | `\vee` |
| $\wedge$ | `\wedge` |
| $\oplus$ | `\oplus` |
| $\ominus$ | `\ominus` |
| $\otimes$ | `\otimes` |
| $\circ$ | `\circ` |
| $\bullet$ | `\bullet` |
| $\diamond$ | `\diamond` |
| $\lhd$ | `\lhd` |
| $\rhd$ | `\rhd` |
| $\unlhd$ | `\unlhd` |
| $\unrhd$ | `\unrhd` |
| $\oslash$ | `\oslash` |
| $\odot$ | `\odot` |
| $\bigcirc$ | `\bigcirc` |
| $\triangleleft$ | `\triangleleft` |
| $\Diamond$ | `\Diamond` |
| $\bigtriangleup$ | `\bigtriangleup` |
| $\bigtriangledown$ | `\bigtriangledown` |
| $\Box$ | `\Box` |
| $\triangleright$ | `\triangleright` |
| $\setminus$ | `\setminus` |
| $\wr$ | `\wr` |
| $\sqrt{x}$ | `\sqrt{x}` |
| $x^{\circ}$ | `x^{\circ}` |
| $\triangledown$ | `\triangledown` |
| $\sqrt[n]{x}$ | `\sqrt[n]{x}` |
| $a^x$ | `a^x` |
| $a^{xyz}$ | `a^{xyz}` |
| $a_x$ | `a_x` |

## 关系符合

| Symbol | Command |
| -- | -- |
| $\le$ | `\le` |
| $\ge$ | `\ge` |
| $\neq$ | `\neq` |
| $\sim$ | `\sim` |
| $\ll$ | `\ll` |
| $\gg$ | `\gg` |
| $\doteq$ | `\doteq` |
| $\simeq$ | `\simeq` |
| $\subset$ | `\subset` |
| $\supset$ | `\supset` |
| $\approx$ | `\approx` |
| $\asymp$ | `\asymp` |
| $\subseteq$ | `\subseteq` |
| $\supseteq$ | `\supseteq` |
| $\cong$ | `\cong` |
| $\smile$ | `\smile` |
| $\sqsubset$ | `\sqsubset` |
| $\sqsupset$ | `\sqsupset` |
| $\equiv$ | `\equiv` |
| $\frown$ | `\frown` |
| $\sqsubseteq$ | `\sqsubseteq` |
| $\sqsupseteq$ | `\sqsupseteq` |
| $\propto$ | `\propto` |
| $\bowtie$ | `\bowtie` |
| $\in$ | `\in` |
| $\ni$ | `\ni` |
| $\prec$ | `\prec` |
| $\succ$ | `\succ` |
| $\vdash$ | `\vdash` |
| $\dashv$ | `\dashv` |
| $\preceq$ | `\preceq` |
| $\succeq$ | `\succeq` |
| $\models$ | `\models` |
| $\perp$ | `\perp` |
| $\parallel$ | `\parallel` |
| $\mid$ | `\mid` |
| $\bumpeq$ | `\bumpeq` |

通过添加 `\n` 可以变成否定关系

| Symbol | Command |
| -- | -- |
| $\nmid$ | `\nmid` |
| $\nleq$ | `\nleq` |
| $\ngeq$ | `\ngeq` |
| $\nsim$ | `\nsim` |
| $\ncong$ | `\ncong` |
| $\nparallel$ | `\nparallel` |
| $\not<$ | `\not<` |
| $\not>$ | `\not>` |
| $\not=$ | `\not= or \neq` |
| $\not\le$ | `\not\le` |
| $\not\ge$ | `\not\ge` |
| $\not\sim$ | `\not\sim` |
| $\not \approx$ | `\not\approx` |
| $\not\cong$ | `\not\cong` |
| $\not\equiv$ | `\not\equiv` |
| $\not\parallel$ | `\not\parallel` |
| $\nless$ | `\nless` |
| $\ngtr$ | `\ngtr` |
| $\lneq$ | `\lneq` |
| $\gneq$ | `\gneq` |
| $\lnsim$ | `\lnsim` |
| $\lneqq$ | `\lneqq` |
| $\gneqq$ | `\gneqq` |

## 希腊字母

### 小写字母

| Symbol | Command |
| -- | -- |
| $\alpha$ | `\alpha` |
| $\beta$ | `\beta` |
| $\gamma$ | `\gamma` |
| $\delta$ | `\delta` |
| $\epsilon$ | `\epsilon` |
| $\varepsilon$ | `\varepsilon` |
| $\zeta$ | `\zeta` |
| $\eta$ | `\eta` |
| $\theta$ | `\theta` |
| $\vartheta$ | `\vartheta` |
| $\iota$ | `\iota` |
| $\kappa$ | `\kappa` |
| $\lambda$ | `\lambda` |
| $\mu$ | `\mu` |
| $\nu$ | `\nu` |
| $\xi$ | `\xi` |
| $\pi$ | `\pi` |
| $\varpi$ | `\varpi` |
| $\rho$ | `\rho` |
| $\varrho$ | `\varrho` |
| $\sigma$ | `\sigma` |
| $\varsigma$ | `\varsigma` |
| $\tau$ | `\tau` |
| $\upsilon$ | `\upsilon` |
| $\phi$ | `\phi` |
| $\varphi$ | `\varphi` |
| $\chi$ | `\chi` |
| $\psi$ | `\psi` |
| $\omega$ | `\omega` |

### 大写字母

| Symbol | Command |
| -- | -- |
| $\Gamma$ | `\Gamma` |
| $\Delta$ | `\Delta` |
| $\Theta$ | `\Theta` |
| $\Lambda$ | `\Lambda` |
| $\Xi$ | `\Xi` |
| $\Pi$ | `\Pi` |
| $\Sigma$ | `\Sigma` |
| $\Upsilon$ | `\Upsilon` |
| $\Phi$ | `\Phi` |
| $\Psi$ | `\Psi` |
| $\Omega$ | `\Omega` |

## 箭头

| Symbol | Command |
| -- | -- |
| $\gets$ | `\gets` |
| $\to$ | `\to` |
| $\leftarrow$ | `\leftarrow` |
| $\Leftarrow$ | `\Leftarrow` |
| $\rightarrow$ | `\rightarrow` |
| $\Rightarrow$ | `\Rightarrow` |
| $\leftrightarrow$ | `\leftrightarrow` |
| $\Leftrightarrow$ | `\Leftrightarrow` |
| $\mapsto$ | `\mapsto` |
| $\hookleftarrow$ | `\hookleftarrow` |
| $\leftharpoonup$ | `\leftharpoonup` |
| $\leftharpoondown$ | `\leftharpoondown` |
| $\rightleftharpoons$ | `\rightleftharpoons` |
| $\longleftarrow$ | `\longleftarrow` |
| $\Longleftarrow$ | `\Longleftarrow` |
| $\longrightarrow$ | `\longrightarrow` |
| $\Longrightarrow$ | `\Longrightarrow` |
| $\longleftrightarrow$ | `\longleftrightarrow` |
| $\Longleftrightarrow$ | `\Longleftrightarrow` |
| $\longmapsto$ | `\longmapsto` |
| $\hookrightarrow$ | `\hookrightarrow` |
| $\rightharpoonup$ | `\rightharpoonup` |
| $\rightharpoondown$ | `\rightharpoondown` |
| $\leadsto$ | `\leadsto` |
| $\uparrow$ | `\uparrow` |
| $\Uparrow$ | `\Uparrow` |
| $\downarrow$ | `\downarrow` |
| $\Downarrow$ | `\Downarrow` |
| $\updownarrow$ | `\updownarrow` |
| $\Updownarrow$ | `\Updownarrow` |
| $\nearrow$ | `\nearrow` |
| $\searrow$ | `\searrow` |
| $\swarrow$ | `\swarrow` |
| $\nwarrow$ | `\nwarrow` |

`\iff` 和 `\implies` 可以代替 `\Longleftrightarrow` 和 `\Longrightarrow`

## 点

| Symbol | Command |
| -- | -- |
| $\cdot$ | `\cdot` |
| $\vdots$ | `\vdots` |
| $\dots$ | `\dots` |
| $\ddots$ | `\ddots` |
| $\cdots$ | `\cdots` |
| $\iddots$ | `\iddots` |

## 音标

| Symbol | Command |
| -- | -- |
| $\hat{x}$ | `\hat{x}` |
| $\check{x}$ | `\check{x}` |
| $\dot{x}$ | `\dot{x}` |
| $\breve{x}$ | `\breve{x}` |
| $\acute{x}$ | `\acute{x}` |
| $\ddot{x}$ | `\ddot{x}` |
| $\grave{x}$ | `\grave{x}` |
| $\tilde{x}$ | `\tilde{x}` |
| $\mathring{x}$ | `\mathring{x}` |
| $\bar{x}$ | `\bar{x}` |
| $\vec{x}$ | `\vec{x}` |

当 `i` 和 `j` 添加音标的时候, 可以使用 `\imath` 和 `\jmath` 避免字母的点干扰到音标

| Symbol | Command |
| -- | -- |
| $\vec{\jmath}$ | `\vec{\jmath}` |
| $\tilde{\imath}$ | `\tilde{\imath}` |

`\tilde` 和 `\hat` 可以使用宽版包括更多的字母

| Symbol | Command |
| -- | -- |
| $\widehat{7+x}$ | `\widehat{7+x}` |
| $\widetilde{abc}$ | `\widetilde{abc}` |

## 其他

| Symbol | Command |
| -- | -- |
| $\infty$ | `\infty` |
| $\triangle$ | `\triangle` |
| $\angle$ | `\angle` |
| $\aleph$ | `\aleph` |
| $\hbar$ | `\hbar` |
| $\imath$ | `\imath` |
| $\jmath$ | `\jmath` |
| $\ell$ | `\ell` |
| $\wp$ | `\wp` |
| $\Re$ | `\Re` |
| $\Im$ | `\Im` |
| $\mho$ | `\mho` |
| $\prime$ | `\prime` |
| $\emptyset$ | `\emptyset` |
| $\nabla$ | `\nabla` |
| $\surd$ | `\surd` |
| $\partial$ | `\partial` |
| $\top$ | `\top` |
| $\bot$ | `\bot` |
| $\vdash$ | `\vdash` |
| $\dashv$ | `\dashv` |
| $\forall$ | `\forall` |
| $\exists$ | `\exists` |
| $\neg$ | `\neg` |
| $\flat$ | `\flat` |
| $\natural$ | `\natural` |
| $\sharp$ | `\sharp` |
| $\backslash$ | `\backslash` |
| $\Box$ | `\Box` |
| $\Diamond$ | `\Diamond` |
| $\clubsuit$ | `\clubsuit` |
| $\diamondsuit$ | `\diamondsuit` |
| $\heartsuit$ | `\heartsuit` |
| $\spadesuit$ | `\spadesuit` |
| $\Join$ | `\Join` |
| $\blacksquare$ | `\blacksquare` |
| $\S$ | `\S` |
| $\P$ | `\P` |
| $\copyright$ | `\copyright` |
| $\pounds$ | `\pounds` |
| $\overarc{ABC}$ | `\overarc{ABC}` |
| $\underarc{XYZ}$ | `\underarc{XYZ}` |
| $\bigstar$ | `\bigstar` |
| $\in$ | `\in` |
| $\cup$ | `\cup` |
| $\square$ | `\square` |
| $\smiley$ | `\smiley` |
| $\mathbb{R}$ | `\mathbb{R} (represents all real numbers)` |
| $\checkmark$ | `\checkmark` |
| $\cancer$ | `\cancer` |

## 命令符

由于部分字符属于关键字, 所以使用以下代替

| Symbol | Command |
| -- | -- |
| $\textdollar$ | `\textdollar or \$` |
| $\&$ | `\&` |
| $\%$ | `\%` |
| $\#$ | `\#` |
| $\_$ | `\_` |
| $\{$ | `\{` |
| $\}$ | `\}` |
| $\backslash$ | `\backslash` |

## 欧洲语言符号

| Symbol | Command |
| -- | -- |
| ${\oe}$ | `{\oe}` |
| ${\ae}$ | `{\ae}` |
| ${\o}$ | `{\o}` |
| ${\OE}$ | `{\OE}` |
| ${\AE}$ | `{\AE}` |
| ${\AA}$ | `{\AA}` |
| ${\O}$ | `{\O}` |
| ${\l}$ | `{\l}` |
| ${\ss}$ | `{\ss}` |
| $\text{!`}$ | !` |
| ${\L}$ | `{\L}` |
| ${\SS}$ | `{\SS}` |

## 括号

默认可以使用括号 ( 和 ), 中括号 [ 和 ], 绝对值 | 和 |. 其他一些符合使用如下

| Symbol | Command |
| -- | -- |
| $\{$ | `\{` |
| $\}$ | `\}` |
| $\|$ | `\|` |
| $\backslash$ | `\backslash` |
| $\lfloor$ | `\lfloor` |
| $\rfloor$ | `\rfloor` |
| $\lceil$ | `\lceil` |
| $\rceil$ | `\rceil` |
| $\langle$ | `\langle` |
| $\rangle$ | `\rangle` |

垂直排版的算式使用简单的符号会显得大小不合适

`(\frac{a}{x} )^2`

得到

$(\frac{a}{x})^2$

这时可以使用 `\left` 和 `\right` 关键字:

`\left(\frac{a}{x} \right)^2`

得到

$\left(\frac{a}{x} \right)^2$

对于方程式或分段函数, 请使用如下 `cases` 环境变量

`f(x) = \begin{cases} x^2 & x \ge 0 \\\\ x & x < 0 \end{cases}`

将会得到

$f(x) = \begin{cases} x^2 & x \ge 0 \\\\ x & x < 0 \end{cases}$

除了 `\left` 和 `\right` 命令, 分数执行地板或天花板功能需要如下使用

`\left\lceil\frac{x}{y}\right\rceil`

和

`\left\lfloor\frac{x}{y}\right\rfloor`

得到 $\left\lceil\frac{x}{y}\right\rceil\text{ and }\left\lfloor\frac{x}{y}\right\rfloor\text{, respectively.}$

使用如下语句

`\underbrace{a_0+a_1+a_2+\cdots+a_n}_{x}`

得到

$\underbrace{a_0+a_1+a_2+\cdots+a_n}_{x}$

或者使用

`\overbrace{a_0+a_1+a_2+\cdots+a_n}^{x}`

得到

$\overbrace{a_0+a_1+a_2+\cdots+a_n}^{x}$

`\left` 和 `\right` 也可以用来调整以下符号的大小

| Symbol | Command |
| -- | -- |
| $\uparrow$ | `\uparrow` |
| $\downarrow$ | `\downarrow` |
| $\updownarrow$ | `\updownarrow` |
| $\Uparrow$ | `\Uparrow` |
| $\Downarrow$ | `\Downarrow` |
| $\Updownarrow$ | `\Updownarrow` |

## 多尺寸符号

| Symbol | Command |
| -- | -- |
| $\sum  \textstyle\sum$ | `\sum` |
| $\int  \textstyle\int$ | `\int` |
| $\oint  \textstyle\oint$ | `\oint` |
| $\prod  \textstyle\prod$ | `\prod` |
| $\coprod  \textstyle\coprod$ | `\coprod` |
| $\bigcap  \textstyle\bigcap$ | `\bigcap` |
| $\bigcup  \textstyle\bigcup$ | `\bigcup` |
| $\bigsqcup  \textstyle\bigsqcup$ | `\bigsqcup` |
| $\bigvee  \textstyle\bigvee$ | `\bigvee` |
| $\bigwedge  \textstyle\bigwedge$ | `\bigwedge` |
| $\bigodot  \textstyle\bigodot$ | `\bigodot` |
| $\bigotimes  \textstyle\bigotimes$ | `\bigotimes` |
| $\bigoplus  \textstyle\bigoplus$ | `\bigoplus` |
| $\biguplus  \textstyle\biguplus$ | `\biguplus` |

## 来源

- [LaTeX:Symbols](https://artofproblemsolving.com/wiki/index.php/LaTeX:Symbols)
