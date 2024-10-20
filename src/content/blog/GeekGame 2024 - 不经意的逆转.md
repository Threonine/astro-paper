---
title: GeekGame 2024 - 不经意的逆转
description: 第一次做到以不经意传输为背景的题目，记录一下
pubDatetime: 2024-10-17T13:41:55+08:00
featured: false
draft: false
tags:
  - Crypto
  - CTF
---
第一次做到以不经意传输（Oblivious transfer）为背景的题目，记录一下. 本题基于 RSA，类似的题目还有

- [zer0ptsCTF 2022 - OK](https://hackmd.io/@theoldmoon0602/SJrf0HPMq)
- [idekCTF 2022 - Decidophobia](https://github.com/EggRoll-Taiyaki/My-CTF-Challenges/tree/main/idekCTF/2022/Decidophobia)

## Flag 1

$$
\begin{cases}
v_0&\equiv{(v-x_0)}^d + {(p+q)}^d+f&\pmod{n}\\
v_1&\equiv{(v-x_1)}^d + {(p-q)}^d+f&\pmod{n}
\end{cases}
$$
两式都有 $f$，考虑作差
$$
v_0-v_1 ={(v-x_0)}^d-{(v-x_1)}^d+{(p+q)}^d-{(p-q)}^d\pmod{n}
$$
由二项式定理
$$
{(p+q)}^d-{(p-q)}^d = 2 \sum_{k \text{ odd}} \binom{d}{k} p^{d-k} q^k = K\cdot q
$$
把模数换到 $q$ 下有
$$
v_0-v_1 ={(v-x_0)}^d-{(v-x_1)}^d\pmod{q}
$$
此时考虑 $v$ 的取值，让右侧尽量简单，尝试
$$
{(v-x_0)}^d =-{(v-x_1)}^d
$$
由于 $d$ 为奇数，那么 $v-x_0=-(v-x_1)$，即 $v=\frac{x_0+x_1}{2}$.

此时有
$$
\begin{align}
v_0-v_1&= 2{(v-x_0)}^d\pmod{q}\\
{(v_0-v_1)}^e&= 2^e(v-x_0)\pmod{q}\\
{(v_0-v_1)}^e&= 2^e(v-x_0)+K'\cdot q\\
K'\cdot q&={(v_0-v_1)}^e-2^e(v-x_0)
\end{align}
$$
计算 $\text{GCD}(K'\cdot q, n)$ 即分解 $n$. 接下来不难求得 $f$.

## Flag 2

$$
\begin{cases}
v_0&\equiv{(v-x_0)}^d + {(p+q)}^d+f&\pmod{n}\\
v_1&\equiv{(v-x_1)}^d + {(p-q)}^d+f^{-1}&\pmod{n}
\end{cases}
$$
 仍取 $v=\frac{x_0+x_1}{2}$，有
$$
\begin{align}
v_0+v_1&\equiv {(v-x_0)}^d+{(v-x_1)}^d+{(p+q)}^d+{(p-q)}^d+f+f^{-1}&\pmod{n}\\
v_0+v_1&\equiv {(v-x_0)}^d+(-1)^d{(v-x_0)}^d+{(p+q)}^d+{(p-q)}^d+f+f^{-1}&\pmod{n}\\
v_0+v_1&\equiv {(p+q)}^d+{(p-q)}^d+f+f^{-1}&\pmod{n}\\
\end{align}
$$
由二项式定理
$$
{(p+q)}^d+{(p-q)}^d = K\cdot p
$$
把模数换到 $p$ 下有
$$
\begin{align}
v_0+v_1&\equiv f+f^{-1}&\pmod{p}\\
f^2-(v_0+v_1)f+1&\equiv 0&\pmod{p}\\
\end{align}
$$
这里 $f$ 是 1024-bit 的，相对于 $p$ 不够小. 可以多取几组数据构成方程组
$$
\begin{cases}
f^2-g_0\cdot f+1&\equiv 0&\pmod{p_0}\\
f^2-g_1\cdot f+1&\equiv 0&\pmod{p_1}\\
&\cdots\\
f^2-g_2\cdot f+1&\equiv 0&\pmod{p_5}\\

\end{cases}
$$
应用多项式的中国剩余定理得到
$$
f^2-g\cdot f+1\equiv 0\pmod{p_0p_1\cdots p_5}\tag 1
$$
注意这里的模数实际上我们是不知道的，不过 [CopperSmith 算法的扩展](https://cryptohack.gitbook.io/cryptobook/lattices/applications/extensions-of-coppersmith-algorithm) 允许我们在模数未知的情况下求小根，也就是对 $(2)$ 式应用 CopperSmith
$$
f^2-g\cdot f+1\equiv 0\pmod{n_0n_1\cdots n_5}\tag 2
$$
能够求出 $(1)$ 式的小根，注意这里令 $\beta=\frac{1}{6}$.

这种解「模单变量多项式方程组」的问题被称为 [SMUPE-problem](https://www.cits.ruhr-uni-bochum.de/imperia/md/content/may/paper/pkc2008.pdf)，[N1CTF 2022-babyecc](https://tanglee.top/2022/11/08/N1CTF-2022-Crypto-Writeups-By-tl2cents/) 也是类似的姿势～
