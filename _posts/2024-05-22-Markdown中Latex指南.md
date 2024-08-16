---
layout: post
title: "Markdown中Latex指南"
categories: Tool
tags: Tool
author: August
math: true
typora-root-url: ..
---

* content
{:toc}

该文主要介绍 `Markdown` 中 `Latex` 使用。



# Markdown中Latex指南



## 1. 常用希腊字母表

| Name       |  Display   | Capital Case |  Display   | Var Case      |    Display    |
| ---------- | :--------: | ------------ | :--------: | ------------- | :-----------: |
| `\alpha`   |  $\alpha$  |              |            |               |               |
| `\beta`    |  $\beta$   |              |            |               |               |
| `\gamma`   |  $\gamma$  | `\Gamma`     |  $\Gamma$  |               |               |
| `\theta`   |  $\theta$  | `\Theta`     |  $\Theta$  | `\vartheta`   |  $\vartheta$  |
| `\mu`      |   $\mu$    |              |            |               |               |
| `\delta`   |  $\delta$  | `\Delta`     |  $\Delta$  |               |               |
| `\epsilon` | $\epsilon$ |              |            | `\varepsilon` | $\varepsilon$ |
| `\sigma`   |  $\sigma$  | `\Sigma`     |  $\Sigma$  | `\varsigma`   |  $\varsigma$  |
| `\pi`      |   $\pi$    | `\Pi`        |   $\Pi$    | `\varpi`      |   $\varpi$    |
| `\omega`   |  $\omega$  | `\Omega`     |  $\Omega$  |               |               |
| `\xi`      |   $\xi$    | `\Xi`        |   $\Xi$    |               |               |
| `\zeta`    |  $\zeta$   |              |            |               |               |
| `\chi`     |   $\chi$   |              |            |               |               |
| `\rho`     |   $\rho$   |              |            | `\varrho`     |   $\varrho$   |
| `\phi`     |   $\phi$   | `\Phi`       |   $\Phi$   | `\varphi`     |   $\varphi$   |
| `\eta`     |   $\eta$   |              |            |               |               |
| `\lambda`  | $\lambda$  | `\Lambda`    | $\Lambda$  |               |               |
| `\kappa`   |  $\kappa$  |              |            |               |               |
| `\nu`      |   $\nu$    |              |            |               |               |
| `\upsilon` | $\upsilon$ | `\Upsilon`   | $\Upsilon$ |               |               |
| `\psi`     |   $\psi$   | `\Psi`       |   $\Psi$   |               |               |
| `\tau`     |   $\tau$   |              |            |               |               |
| `\iota`    |  $\iota$   |              |            |               |               |
| `o`        |    $o$     |              |            |               |               |



## 2. 常用特殊字符表

| Name         |   Display    | Name         |   Display    | Name        |   Display   | Name       |  Display   |
| ------------ | :----------: | ------------ | :----------: | ----------- | :---------: | ---------- | :--------: |
| `\times`     |   $\times$   | `\div`       |    $\div$    | `\pm`       |    $\pm$    | `\mp`      |   $\mp$    |
| `\otimes`    |  $\otimes$   | `\ominus`    |  $\ominus$   | `\oplus`    |  $\oplus$   | `\odot`    |  $\odot$   |
| `\oslash`    |  $\oslash$   | `\triangleq` | $\triangleq$ | `\ne`       |    $\ne$    | `\equiv`   |  $\equiv$  |
| `\lt`        |    $\lt$     | `\gt`        |    $\gt$     | `\le`       |    $\le$    | `\ge`      |   $\ge$    |
| `\cup`       |    $\cup$    | `\cap`       |    $\cap$    | `\Cup`      |   $\Cup$    | `\Cap`     |   $\Cap$   |
| `\bigcup`    |  $\bigcup$   | `\bigcap`    |  $\bigcap$   | `\ast`      |   $\ast$    | `\star`    |  $\star$   |
| `\bigotimes` | $\bigotimes$ | `\bigoplus`  | $\bigoplus$  | `\circ`     |   $\circ$   | `\bullet`  | $\bullet$  |
| `\bigcirc`   |  $\bigcirc$  | `\amalg`     |   $\amalg$   | `\to`       |    $\to$    | `\infty`   |  $\infty$  |
| `\vee`       |    $\vee$    | `\wedge`     |   $\wedge$   | `\lhd`      |   $\lhd$    | `\rhd`     |   $\rhd$   |
| `\bigvee`    |  $\bigvee$   | `\bigwedge`  | $\bigwedge$  | `\unlhd`    |  $\unlhd$   | `\unrhd`   |  $\unrhd$  |
| `\sqcap`     |   $\sqcap$   | `\sqcup`     |   $\sqcup$   | `\prec`     |   $\prec$   | `\succ`    |  $\succ$   |
| `\subset`    |  $\subset$   | `\supset`    |  $\supset$   | `\sim`      |   $\sim$    | `\approx`  | $\approx$  |
| `\subseteq`  | $\subseteq$  | `\supseteq`  | $\supseteq$  | `\cong`     |   $\cong$   | `\doteq`   |  $\doteq$  |
| `\setminus`  | $\setminus$  | `\mid`       |    $\mid$    | `\ll`       |    $\ll$    | `\gg`      |   $\gg$    |
| `\parallel`  | $\parallel$  | `\Join`      |   $\Join$    | `\in`       |    $\in$    | `\notin`   |  $\notin$  |
| `\propto`    |  $\propto$   | `\neg`       |    $\neg$    | `\ldots`    |  $\ldots$   | `\cdots`   |  $\cdots$  |
| `\forall`    |  $\forall$   | `\exists`    |  $\exists$   | `\vdots`    |  $\vdots$   | `\ddots`   |  $\ddots$  |
| `\aleph`     |   $\aleph$   | `\nabla`     |   $\nabla$   | `\imath`    |  $\imath$   | `\jmath`   |  $\jmath$  |
| `\ell`       |    $\ell$    | `\partial`   |  $\partial$  | `\int`      |   $\int$    | `\oint`    |  $\oint$   |
| `\uplus`     |   $\uplus$   | `\biguplus`  | $\biguplus$  | `\boxtimes` | $\boxtimes$ | `\boxplus` | $\boxplus$ |



## 3. 其他

| Name                 |       Display        | Name                  |        Display        |
| -------------------- | :------------------: | --------------------- | :-------------------: |
| `\triangleleft`      |   $\triangleleft$    | `\triangleright`      |   $\triangleright$    |
| `\bigtriangleup`     |   $\bigtriangleup$   | `\bigtriangledown`    |  $\bigtriangledown$   |
| `\uparrow`           |      $\uparrow$      | `\downarrow`          |     $\downarrow$      |
| `\leftarrow`         |     $\leftarrow$     | `\rightarrow`         |     $\rightarrow$     |
| `\Leftarrow`         |     $\Leftarrow$     | `\Rightarrow`         |     $\Rightarrow$     |
| `\longleftarrow`     |   $\longleftarrow$   | `\longrightarrow`     |   $\longrightarrow$   |
| `\Longleftarrow`     |   $\Longleftarrow$   | `\Longrightarrow`     |   $\Longrightarrow$   |
| `\leftrightarrow`    |  $\leftrightarrow$   | `\longleftrightarrow` | $\longleftrightarrow$ |
| `\Leftrightarrow`    |  $\Leftrightarrow$   | `\Longleftrightarrow` | $\Longleftrightarrow$ |
| `\leftharpoonup`     |   $\leftharpoonup$   | `\rightharpoonup`     |   $\rightharpoonup$   |
| `\leftharpoondown`   |  $\leftharpoondown$  | `\rightharpoondown`   |  $\rightharpoondown$  |
| `\rightleftharpoons` | $\rightleftharpoons$ | `\S`                  |         $\S$          |
| `\nwarrow`           |      $\nwarrow$      | `\nearrow`            |      $\nearrow$       |
| `\swarrow`           |      $\swarrow$      | `\searrow`            |      $\searrow$       |
| `\triangle`          |     $\triangle$      | `\box`                |        $\Box$         |
| `\diamond`           |      $\diamond$      | `\diamondsuit`        |    $\diamondsuit$     |
| `\heartsuit`         |     $\heartsuit$     | `\clubsuit`           |      $\clubsuit$      |
| `\spadesuit`         |     $\spadesuit$     |                       |                       |



## 4. 公式语法

- 上下标`_ ^ , _{}^{}`：
    $$ y = x_i^{a_1^2} $$

- 公式中插入空格`\,  \;  \quad  \qquad`间隔依次变宽：
    $$ ab $$
    $$ a\,b $$
    $$ a\;b $$
    $$ a\quad b $$
    $$ a\qquad b $$

- 字母上方横线`\overline{}, \bar{}`：
    $$ \overline{xyz} 或 \bar{x} $$

- 字母下方横线`\underline{}`：
    $$ \underline{ABC} $$

- 字母上方波浪线`\tilde{}, \widetilde{}`：
    $$ \tilde{A} 或 \widetilde{ABC} $$

- 字母上方尖号^`\hat{}, \widehat{}`：
    $$ \hat{A} 或 \widehat{ABC} $$

- 字母上方箭头`\vec{}, \overleftarrow{}, \overrightarrow{}`：
    $$ \vec{ab} 或 \overleftarrow{ab} 或 \overrightarrow{ab} $$

- 字母上方花括号`\overbrace{}`，或下方花括号`\underbrace{}`：
    $$ \overbrace{1+2+3} 或 \underbrace{1+2+3} $$

- 字母上方点号`\dot{}, \ddot{}`：
    $$ \dot{a} 或 \ddot{a} $$
    
- 省略号`\dots, \cdots`
    $$ 1,2,\dots  \qquad  1,2,\cdots $$

- 积分`\int_{}^{}`：
    $$ \int_{-\infty}^{+\infty} f(x) \mathrm{d}x $$

- 双重积分`\iint`：
    $$ \iint_{-\infty}^{+\infty} f(x,y) \mathrm{d}x \mathrm{d}y $$

- 行内积分 `\int_{-\infty}^{+\infty} f(x) \mathrm{d}x`：
    $$ \int_{-\infty}^{+\infty} f(x) \mathrm{d}x $$

- 行内积分limits模式 `\int\limits_{}^{}`：
    $$ \int\limits_{-\infty}^{+\infty} f(x) \mathrm{d}x $$

- 行内积分display模式 `\displaystyle \int_{}^{}`：
    $$ \displaystyle \int_{-\infty}^{+\infty} f(x) \mathrm{d}x $$

- 圆圈积分 `\oint`：
    $$ \oint_{-\infty}^{+\infty} $$

- 求和`\sum_{}^{}`：
    $$ \sum_{i=1}^{n} i^2 $$

- 行内求和：
    $$ \sum_{i=1}^{n} i^2 $$

- 行内求和limits模式 `\sum\limits_{}^{}`：
    $$ \sum\limits_{i=1}^{n} i^2 $$

- 行内求和display模式 `\displaystyle \sum_{}^{}`：
    $$ \displaystyle \sum_{i=1}^{n} i^2 $$

- 求乘积 `\prod_{}^{}`：
    $$ \prod_{i=1}^{n} a_i $$

- 分数 `\frac{up}{down}`：
    $$ x_1,x_2 = \frac{b^2 \pm 4ac}{2a} $$

- 根号 `\sqrt`：
    $$ r = \sqrt{x^2+y^2} $$

- 多次根号 `\sqrt[n]`：
    $$ x^{2/3} = \sqrt[3]{x^2} $$



## 5. 方程组

- 如果各个方程需要在某个字符处对齐（如等号对齐），只需在所有要对齐的字符前加上 `&` 符号，取消也是 `&` 符号 。如果不需要公式编号，只需在宏包名称后加上 `*` 号。

### 5.1. 常见方程组

```latex
\begin{aligned}
    \left\{
        \begin{aligned}
            2x + y &= 1  \\
            2x + 2y &= 2
        \end{aligned}
    \right.
\end{aligned}
```

$$
\begin{aligned}
    \left\{
        \begin{aligned}
            2x + y &= 1 \\
            2x + 2y &= 2
        \end{aligned}
    \right.
\end{aligned}
$$

### 5.2. 分类方程组

```latex
f(x) =
\begin{cases}
    x^2 \qquad & a \gt 0 \\
    e^x \qquad & a \le 0
\end{cases}
```

$$
f(x) = \begin{cases}
    x^2 \qquad & a \gt 0 \\
    e^x \qquad & a \le 0
\end{cases}
$$


### 5.3. 对齐


```latex
$$
\begin{aligned}
    F_{0} &= (A \oplus B \oplus C) &&\oplus D &&\oplus E \\
    F_{1} &= (A \oplus B) &&\oplus (C \oplus D) &&\oplus E \\
    F_{2} &= A &&\oplus (B \oplus C \oplus D) &&\oplus E \\
    F_{3} &= A &&\oplus (B \oplus C) &&\oplus (D \oplus E)
\end{aligned}
$$
```

$$
\begin{aligned}
    F_{0} &= (A \oplus B \oplus C) &&\oplus D &&\oplus E \\
    F_{1} &= (A \oplus B) &&\oplus (C \oplus D) &&\oplus E \\
    F_{2} &= A &&\oplus (B \oplus C \oplus D) &&\oplus E \\
    F_{3} &= A &&\oplus (B \oplus C) &&\oplus (D \oplus E)
\end{aligned}
$$



## 6. 矩阵

在开头使用 `begin{matrix}`，在结尾使用 `end{matrix}`，在中间插入矩阵元素，每个元素之间插入 `&` ，并在每行结尾处使用 `\\` 。

常见矩阵：`matrix`、`pmatrix`、`bmatrix`、`Bmatrix`、`vmatrix`、`Vmatrix`。

### 6.1. 无框矩阵

```latex
$$
\begin{matrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9
\end{matrix}
$$
```

$$
\begin{matrix}
1 & 2 & 3 \\
4 & 5 & 6 \\
7 & 8 & 9
\end{matrix}
$$

### 6.2. 边框矩阵

```latex
$$
\begin{matrix}
    1 & 2 \\
    3 & 4 \\
\end{matrix}
$$

$$
\begin{pmatrix}
    1 & 2 \\
    3 & 4 \\
\end{pmatrix}
$$

$$
\begin{bmatrix}
    1 & 2 \\
    3 & 4 \\
\end{bmatrix}
$$

$$
\begin{Bmatrix}
    1 & 2 \\
    3 & 4 \\
\end{Bmatrix}
$$

$$
\begin{vmatrix}
    1 & 2 \\
    3 & 4 \\
\end{vmatrix}
$$

$$
\begin{Vmatrix}
    1 & 2 \\
    3 & 4 \\
\end{Vmatrix}
$$
```

$$
\begin{matrix}
    1 & 2 \\
    3 & 4 \\
\end{matrix}
$$

$$
\begin{pmatrix}
    1 & 2 \\
    3 & 4 \\
\end{pmatrix}
$$

$$
\begin{bmatrix}
    1 & 2 \\
    3 & 4 \\
\end{bmatrix}
$$

$$
\begin{Bmatrix}
    1 & 2 \\
    3 & 4 \\
\end{Bmatrix}
$$

$$
\begin{vmatrix}
    1 & 2 \\
    3 & 4 \\
\end{vmatrix}
$$

$$
\begin{Vmatrix}
    1 & 2 \\
    3 & 4 \\
\end{Vmatrix}
$$


### 6.3. 带省略符号的矩阵

```latex
$$
A_{m,n} = 
\begin{pmatrix}
    a_{1,1} & a_{1,2} & \cdots & a_{1,n} \\
    a_{2,1} & a_{2,2} & \cdots & a_{2,n} \\
    \vdots & \vdots & \ddots & \vdots \\
    a_{m,1} & a_{m,2} & \cdots & a_{m,n} 
\end{pmatrix}
$$
```

$$
A_{m,n} = 
\begin{pmatrix}
    a_{1,1} & a_{1,2} & \cdots & a_{1,n} \\
    a_{2,1} & a_{2,2} & \cdots & a_{2,n} \\
    \vdots & \vdots & \ddots & \vdots \\
    a_{m,1} & a_{m,2} & \cdots & a_{m,n} 
\end{pmatrix}
$$



## 7. 数组和表格

数组和表格均以 `begin{array}` 开头，并在其后定义列数及每一列的文本对齐属性，`c` `l` `r` 分别代表居中、左对齐及右对齐。若需要插入垂直分割线，在定义式中插入 `|` ，若要插入水平分割线，在下一行输入前插入 `\hline` 。

### 7.1. 数组

```latex
$$
\left[
    \begin{array}{cc|c}
      1&2&3\\
      4&5&6
    \end{array}
\right]
$$
```

$$
\left[
    \begin{array}{cc|c}
      1&2&3\\
      4&5&6
    \end{array}
\right]
$$

### 7.2. 表格

```latex
\begin{array}{c|lcr}
    n & \text{左对齐} & \text{居中对齐} & \text{右对齐} \\
    \hline
    1 & 0.24 & 1 & 125 \\
    2 & -1 & 189 & -8 \\
    3 & -20 & 2000 & 1+10i
\end{array}
```

$$
\begin{array}{c|lcr}
    n & \text{左对齐} & \text{居中对齐} & \text{右对齐} \\
    \hline
    1 & 0.24 & 1 & 125 \\
    2 & -1 & 189 & -8 \\
    3 & -20 & 2000 & 1+10i
\end{array}
$$



## 8. 字体大小

```latex
$\Huge Hello!$
$\huge Hello!$
$\LARGE Hello!$
$\Large Hello!$
$\large Hello!$
$\normalsize Hello!$
$\small Hello!$
$\scriptsize Hello!$
$\tiny Hello!$
```

$\Huge Hello!$

$\huge Hello!$

$\LARGE Hello!$

$\Large Hello!$

$\large Hello!$

$\normalsize Hello!$

$\small Hello!$

$\scriptsize Hello!$

$\tiny Hello!$



