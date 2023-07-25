---
title: MarkDown数学公式
date: 2023-07-19 17:44:00 +0800
tags: [markdown]
---
## 行内与独行

- 行内公式

$xyz$

- 独行公式

$$公式内容$$

## 上标，下标，组合

|||
|-|-|
|上标,例如指数|$x^4$|
|下标|$x_{10}$|
|组合|${16}_{8}O{2+}_{2}$|

## 汉字，字体，格式

|||
|-|-|
|汉字形式||
|字体大小控制|$\frac{x+y}{y+z}$<br>$\displaystyle \frac{x+y}{y+z}$|
|下划线|$\underline{x+y}$|
|标签|$$a=x^2-y^3\tag{1}$$|
|上大括号|$\overbrace{a+b+c+d}^{2.0}$|
|下大括号|$a+\underbrace{b+c}_{1.0}+d$|
|上位符号|$\vec{x}\stackrel{\mathrm{def}}{=}{x_1,\dots,x_n}$|

## 占位符

|||
|-|-|
|两个quad空格|$x\qquad y$|
|quad空格|$x\quad y$|
|大空格|$x\ y$|
|中空格|$x\: y$|
|小空格|$x\, y$|
|没有空格|$xy$|
|紧贴|$x\! y$|

## 定界符与组合

|||
|-|-|
|括号|$（）\big(\big) \Big(\Big) \bigg(\bigg) \Bigg(\Bigg)$|
|中括号|$[x+y]$|
|大括号|$\{x+y\}$|
|自适应括号|$\left(x\right)$<br>$\left(x{yz}\right)$|
|组合公式|${n+1 \choose k}={n \choose k}+{n \choose k-1}$|

## 四则运算

|||
|-|-|
|加法|$x+y=z$|
|减法|$x-y=z$|
|加减|$x\pm y=z$|
|减加|$x\mp y=z$|
|叉乘|$x\times y=z$|
|点乘|$x\cdot y=z$|
|星乘|$x \ast y=z$|
|除法|$x \div y=z$|
|斜除|$x/y=z$|
|分式|$\frac{x+y}{y+z}$<br>${x+y}\over{y+z}$|
|绝对值|$\|x+y\|$|

## 高级运算

|||
|-|-|
|平均数|$\overline{x}$|
|开二次方|$\sqrt{x11}$|
|开方|$\sqrt[3]{x+y}$|
|对数|$\log_3{(x)}$|
|极限|$\lim^{x \to \infty}_{y \to 0}{\frac{x}{y}}$<br>$\displaystyle \lim^{x \to \infty}_{y \to 0}{\frac{x}{y}}$|
|求和|$\sum^{x \to \infty}_{y \to 0}{\frac{x}{y}}$<br>$\displaystyle \sum^{x \to \infty}_{y \to 0}{\frac{x}{y}}$|
|积分|$\int^{\infty}_{0}{xdx}$<br>$\displaystyle \int^{\infty}_{0}{xdx}$|
|微分|$\frac{\partial x}{\partial y}$|

### 矩阵

使用`\begin{matrix}…\end{matrix}`这样的形式来表示矩阵，在`\begin` 与`\end`之间加入矩阵中的元素即可。矩阵的行之间使用`\\`分隔，列之间使用`&`分隔，例如:
$$
\begin{matrix}
1 & x & x^2 \\
1 & y & y^2 \\
1 & z & z^2 \\
\end{matrix}
$$

## 逻辑运算

|||
|-|-|
|等于|$x+y=z$|
|大于|$x+y>z$|
|小于|$x+y<z$|
|大于等于|$x+y\geq z$|
|小于等于|${x+y}\leq{z}$|
|不等于|$x+y \neq z$|
|不大于等于|$x+y \ngeq z$<br>$x+y \not\geq z$|
|不小于等于|$x+y \nleq z$<br>$x+y \not\leq z$|
|约等于|$x+y \approx z$|
|恒等于|$x+y \equiv z$|

## 集合运算

|||
|-|-|
|属于|$x \in y$|
|不属于|$x \in y$|
|子集|$x \subset y$<br>$x \supset y$|
|真子集|$x \subseteq y$<br>$x \supseteq y$|
|非真子集|$x \subsetneq y$<br>$x \supsetneq y$|
|非子集|$x \not\subset y$<br>$x \not\supset y$|
|并集|$x \cup y$|
|交集|$x \cap y$|
|差集|$x \setminus y$|
|同或|$x \bigodot y$|
|同与|$x \bigotimes y$|
|实数|$\mathbb{R}$|
|自然数|$\mathbb{Z}$|
|空集|$\emptyset$|

## 数学符号

|效果|实现|
|-|-|
|无穷|$\infty$|
|虚数|$\imath$<br>$\imath$|
|数学符号|$\hat{a}$<br>$\check{a}$<br>$\breve{a}$<br>$\tilde{a}$<br>$\bar{a}$<br>$\acute{a}$<br>$\grave{a}$<br>$\mathring{a}$|
|矢量符号|$\vec{a}$|
|一阶导数符号|$\dot{a}$|
|二阶导数符号|$\ddot{a}$|
|上箭头|$\uparrow$<br>$\Uparrow$|
|下箭头|$\downarrow$<br>$\Downarrow$|
|左箭头|$\leftarrow$<br>$\Leftarrow$|
|右箭头|$\rightarrow$<br>$\Rightarrow$|
|底端对齐的省略号|$1,2,\ldots,n$|
|中线对齐的省略号|$x_1^2 + x_2^2 + \cdots + x_n^2$|
|竖直对齐的省略号|$\vdots$|
|斜对齐的省略号|$\ddots$|

## 希腊字母

|大写字母|实现|小写字母|实现|
|-|-|-|-|
|A|$A$|α|$\alpha$|
|B|$B$|β|$\beta$|
|Γ|$\Gamma$|γ|$\gamma$|
|Δ|$\Delta$|δ|$\delta$|
|E|$E$|ϵ|$\epsilon$|
|Z|$Z$|ζ|$\zeta$|
|H|$H$|η|$\eta$|
|Θ|$\Theta$|θ|$\theta$|
|I|$I$|ι|$\iota$|
|K|$K$|κ|$\kappa$|
|Λ|$\Lambda$|λ|$\lambda$|
|M|$M$|μ|$\mu$|
|N|$N$|ν|$\nu$|
|Ξ|$\Xi$|ξ|$\xi$|
|O|$O$|ο|$\omicron$|
|Π|$\Pi$|π|$\pi$|
|P|$P$|ρ|$\rho$|
|Σ|$\Sigma$|σ|$\sigma$|
|T|$T$|τ|$\tau$|
|Υ|$\Upsilon$|υ|$\upsilon$|
|Φ|$\Phi$|ϕ|$\phi$|
|X|$X$|χ|$\chi$|
|Ψ|$\Psi$|ψ|$\psi$|
|Ω|$\v{10}$|ω|$\omega$|
