---
title: 单变量 CopperSmith 学习
description: 准备补补多变量 CopperSmith，先复习一下单变量
pubDatetime: 2024-11-22T23:43:39+08:00
featured: false
draft: false
tags:
  - "#Crypto"
  - "#CTF"
---
给定一个模多项式方程 $F(x)\equiv 0\pmod{M}$，有哪些解法呢？

首先我们知道，如果我们知道模数的分解，模多项式方程是容易解的. 实践中我们用 SageMath 的 `f.roots()` 就可以解决.

而如果分解模数很困难，模多项式方程是不存在一个通用的解法的，这也很好理解：假设我们能解方程 $x^2\equiv1\pmod{M}$ 的非平凡解，就有 $M\mid (x+1)(x-1)$，非平凡解意味着存在着非平凡因子，我们只要计算 GCD 就能分解 M 了. 这就和「分解模数很困难」的假设矛盾了.

如果不存在模 M 的操作，使用数值分析的方法（如牛顿迭代法）也可以容易地求解方程. 关键就是这个模 M 的操作，让多项式的值「溢出」了，从而使得方程变得不可解，那么有没有可能让这个操作不存在呢？没有条件就创造条件，考虑一个很特殊的情况：度为 $d$ 的首一多项式
$$
F(x)=x^d+a_{d-1}x^{d-1}+\cdots +a_1x+a_0
$$
如果我们要找的这个 $x_0$ 足够小，满足 $|x_0|<M^{1/d}$，同时多项式的系数又足够小，那么 $F(x_0)$ 的值可能根本就不会超过 M，模 M 的操作可以看作不存在，这个方程就是可解的了.

多小才算小呢？

可以把多项式用一个行向量来表示，例如多项式 $F(x)$ 的行向量表示：
$$
b_F=(a_0,a_1X,a_2X^2,\cdots a_dX^d)
$$
（特别注意这个地方不是直接取系数，其实可以写成 $g(xX)$ 这样不容易搞错）

**Howgrave-Graham 定理** 给了我们一个上界：
$$
\text{If }\Vert b_F\Vert< \frac{M}{\sqrt{d+1}}\text{, then }F(x_0)=0
$$
问题是，很多时候我们只能保证要找的 $x_0$ 小，「多项式的系数足够小」这个条件是不满足的.

CopperSmith 给出了一个方法，我们只需保证要找的 $x_0$ 小，就能从要解的 $F(x)$ 去构造出另一个多项式 $G(x)$，这个 $G(x)$ 满足「多项式的系数又足够小」这个条件！

## 问题形式

首一（$a_d=1$）多项式 $F(x)=\sum_{i=0}^da_ix^i\in \mathbb{Z}[x]$

我们想要求方程 $F(x)\equiv 0\pmod{M}$ 的根 $x_0\in \mathbb{Z}$，满足 $x_0<|X|$.

为什么首一？如果不是首一

- 若 $\text{gcd}(a_d,M)=1$，直接乘上逆元 $a_d\pmod{M}$ 就变成首一
- 若 $\text{gcd}(a_d,M)\gt 1$，则可以部分分解 $M$

## 构造 $G(x)$ —— 简单的例子

考虑多项式
$$
F(x)=x^2+8x+2\equiv 0\pmod{11}
$$
现在它最大的系数是 8，我们想让它的系数变小，该怎么做？
我们取
$$
F_1(x)=-F(x)=x^2-3x+2\equiv 0\pmod{11}
$$
现在它最大的系数是 3.
还能更小吗？我们取
$$
F_2(x)=xF_1(x)=x^3-3x^2+2x\equiv 0\pmod{11}
$$
这样就有
$$
G(x)=F_1(x)+F_2(x)=x^3-2x^2-x+2\equiv 0\pmod{11}
$$
以上过程展示了让系数变小的思路： 我们可以想办法写出「各种在模 $M$ 下为 0 的多项式」，将它们进行**线性组合**来得到 $G(x)$，线性组合的结果仍然为 0.

注意到这里的要求是要求系数「足够小」，所以我们可以构造格，用 LLL 算法来求解.

在这之前，「各种在模 $M$ 下为 0 的多项式」究竟如何程序化构造呢？接下来先考虑最简单的一种：

## 构造 $G(x)$ —— Håstad 的方法

由于 $F(x)$ 是 $d$ 次的**首一**多项式，我们用线性组合让除了首项（也就是 $d$ 次项）的其他项的系数变小. **最简单的想法就是直接取 $d-1$ 个单项式**，让它们的系数都为 $M$，也就是
$$
G_i(x)=Mx^i
$$
我们把 $G_i(x)$ 和 $F(x)$ 写为行向量表示，拼成一个格基：
$$
B =
\begin{pmatrix}
M & 0 & \cdots & 0 & 0 \\
0 & MX & \cdots & 0 & 0 \\
\vdots & \vdots & \ddots & \vdots & \vdots \\
0 & 0 & \cdots & MX^{d-1} & 0 \\
a_0 & a_1 X & \cdots & a_{d-1}X^{d-1} & X^d
\end{pmatrix}
$$
对其使用 LLL 算法，规约后的第 0 行结果就是我们要的 $G(x)$ 的向量表示了.

### LLL 算法

LLL 算法可以在多项式时间内，从给定的格基找到短向量满足
$$
\Vert\underline{b_1}\Vert\leq 2^{(n-1)/4}\det(L)^{1/n}
$$
还记得上面说的**Howgrave-Graham 定理**吗？为了满足它，我们有
$$
\Vert\underline{b_1}\Vert\leq 2^{(n-1)/4}\det(L)^{1/n}<\frac{N^m}{\sqrt{n}}
$$
我们会看到，在 CopperSmith 方法中，当 $M$ 充分大的时候，对于 $m\approx \log N$ 有 $\det(L$) $2^{(n-1)/4}$ 和 $\frac{M}{\sqrt{n}}$ 可以忽略. 所以有**启动条件**：
$$
\det(L)\leq M^n=M^{d+1}
$$
当格的行列式足够小的时候：CopperSmith，启动！

换句话说，我们选多项式的依据，就使得它们对应的向量形式所张成的格的**行列式尽可能小**.

## CopperSmith - 达到 $N^\frac{1}{d}$

关于 CopperSmith 方法，几篇 paper 的定义都有所出入，`small_roots` 是参考 [*New RSA Vulnerabilities Using Lattice Reduction Methods*](https://d-nb.info/972386416/34) 来的，就参考这个来总结.

这里的定义补充了一种特别常见的情形，就是对于 $N$ 的一个因子 $b\geq N^\beta$，我们不知道 $b$，但是想求模数为 $b$ 的多项式方程的小根.

这里要注意的是，你在搜索引擎搜索 `small_roots` 函数很可能会搜到这篇先知文章[Coppersmith算法解决RSA加密问题](https://xz.aliyun.com/t/13769)，里面对 `small_roots` 的讲解完全是在扯淡.

**CopperSmith 方法**：$N$ 是一个分解未知的整数，有一个因子 $b\geq N^\beta$. 令 $F(x)$ 为一个次数为 $\delta$ 的首一多项式，其在模 $N$ 下至少有一个小根 $x_0$，满足 $|x_0| \leq \frac{1}{2}N^{\frac{\beta^2}{\delta} - \epsilon}$. 那么，可以在多项式时间内（关于 $\delta$、$1/\epsilon$ 和 $\log(M)$）找到 $x_0$.

`small_roots` 除了多项式本身，还有三个可选参数 `X`, `beta`, `epsilon`

- $X$ 就是小根的上界，在这个定义里是令 $X=\frac{1}{2}N^{\frac{\beta^2}{d} - \epsilon}$ 的，不过在 SageMath 里也允许自己指定
- $\beta$ 表示因子的大小，如果不指定就是 1.0
- $\epsilon$  是用来控制精度的，如果不指定就是 $\frac{1}{8}\beta$. 这个值取得越小就越能解出结果，但是矩阵维数会相应增大，导致 LLL 变慢.

根据这些参数计算参数 `m` 和 `t`
$$
\begin{align}
m&\geq \max \left\{\frac{\beta^2}{\delta\epsilon},\frac{7\beta}{\delta}  \right\}\\
t&=\left\lfloor \delta m(\frac{1}{\beta}-1)\right\rfloor
\end{align}
$$

从而选择许多个多项式，进而构造格基矩阵. 这里面的细节我们不追究.

但是，选多项式的策略是从何而来？前面说到，我们想让行列式尽可能小，如何相应地改变选取「各种在模 $M$ 下为 0 的多项式」的策略呢？这就是 CopperSmith 的核心科技了：

- **策略 1**：注意到 $x^iF(x)$ 在模 $M$ 下仍然是等于 0 的. 这被称为 "x-shift" polynomial
- **策略 2**：注意到 $F(x)^k$ 在模 $M^k$ 下仍然是等于 0 的（注意模数变了）

策略 1 产生了更多的多项式，增大了 $d$；策略 2 增大了模数 $M$，这都能扩大启动条件右侧的界，这样行列式就更容易符合启动条件了.

【例子】令 $p = 2^{30} + 3, q = 2^{32} + 15, M = pq$，考虑多项式
$$
\begin{align}
F (x) &= a_0 + a_1x + a_2x^2 + a_3x^3\\
&= 1942528644709637042 + 1234567890123456789x + 987654321987654321x^2 + x^3
\end{align}
$$
我们已知 $|x_0|\leq2^{14}$，设 $X=2^{14}$，注意到 $X\approx M^{1/4.4}$，使用基础方法得不到解

这里我们令 $m=2$，也就是让所有多项式在模 $N^m=N^2$ 下面有根 $x_0$（注意这边有些博客文章说的不准确）. 取七个多项式 $N^2,N^2x,N^2x^2,Nf(x),xNf(x),x^2Nf(x),f^2(x)$，构造如下格基矩阵
$$
B=\begin{pmatrix}
    M^2 & 0 & 0 & 0 & 0 & 0 & 0 \\
    0 & M^2X & 0 & 0 & 0 & 0 & 0 \\
    0 & 0 & M^2X^2 & 0 & 0 & 0 & 0 \\
    Ma_0 & Ma_1X & Ma_2X^2 & MX^3 & 0 & 0 & 0 \\
    0 & Ma_0X & Ma_1X^2 & Ma_2X^3 & MX^4 & 0 & 0 \\
    0 & 0 & Ma_0X^2 & Ma_1X^3 & Ma_2X^4 & MX^5 & 0 \\
    a_0^2 & 2a_0a_1X & (a_1^2 + 2a_0a_2)X^2 & (2a_0 + 2a_1a_2)X^3 & (a_2^2 + 2a_1)X^4 & 2a_2X^5 & X^6
\end{pmatrix}
$$
$\det(L)=|\det(B)|=N^9X^{21}$，根据启动条件，$N^9X^{21}\leq N^{2\cdot 7}=N^{14}$，因此 $X\leq N^{\frac{5}{21}}$

## flatter 加速板子

由于 CopperSmith 最耗时的步骤仍然是 LLL，我们可以用 flatter 去做加速

```python
from re import findall  
from subprocess import check_output  

def flatter(M):  
    # compile https://github.com/keeganryan/flatter and put it in $PATH  
    z = "[[" + "]\n[".join(" ".join(map(str, row)) for row in M) + "]]"  
    ret = check_output(["flatter"], input=z.encode())  
    return matrix(M.nrows(), M.ncols(), map(int, findall(b"-?\\d+", ret)))  
  
  
def small_roots(self, X=None, beta=1.0, epsilon=None, **kwds):  
    from sage.misc.verbose import verbose  
    from sage.matrix.constructor import Matrix  
    from sage.rings.real_mpfr import RR  
  
    N = self.parent().characteristic()  
  
    if not self.is_monic():  
        raise ArithmeticError("Polynomial must be monic.")  
  
    beta = RR(beta)  
    if beta <= 0.0 or beta > 1.0:  
        raise ValueError("0.0 < beta <= 1.0 not satisfied.")  
  
    f = self.change_ring(ZZ)  
  
    P, (x,) = f.parent().objgens()  
  
    delta = f.degree()  
  
    if epsilon is None:  
        epsilon = beta / 8  
    verbose("epsilon = %f" % epsilon, level=2)  
  
    m = max(beta**2 / (delta * epsilon), 7 * beta / delta).ceil()  
    verbose("m = %d" % m, level=2)  
  
    t = int((delta * m * (1 / beta - 1)).floor())  
    verbose("t = %d" % t, level=2)  
  
    if X is None:  
        X = (0.5 * N ** (beta**2 / delta - epsilon)).ceil()  
    verbose("X = %s" % X, level=2)  
  
    # we could do this much faster, but this is a cheap step  
    # compared to LLL  
    g = [x**j * N ** (m - i) * f**i for i in range(m) for j in range(delta)]  
    g.extend([x**i * f**m for i in range(t)])  # h  
  
    B = Matrix(ZZ, len(g), delta * m + max(delta, t))  
    for i in range(B.nrows()):  
        for j in range(g[i].degree() + 1):  
            B[i, j] = g[i][j] * X**j  
  
    B = flatter(B)  
  
    f = sum([ZZ(B[0, i] // X**i) * x**i for i in range(B.ncols())])  
    R = f.roots()  
  
    ZmodN = self.base_ring()  
    roots = set([ZmodN(r) for r, m in R if abs(r) <= X])  
    Nbeta = N**beta  
    return [root for root in roots if N.gcd(ZZ(self(root))) >= Nbeta]
```
