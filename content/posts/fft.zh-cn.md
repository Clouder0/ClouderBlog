---
title: "快速傅立叶变换学习笔记"
date: 2021-03-27T14:54:26+08:00

---


# 快速傅里叶变换

**离散傅里叶变换（Discrete Fourier Transform，缩写为 DFT）**，是傅里叶变换在时域和频域上都呈离散的形式，将信号的时域采样变换为其 DTFT 的频域采样。

**FFT** 是一种高效实现 DFT 的算法，称为 **快速傅立叶变换（Fast Fourier Transform，FFT）**。它对傅里叶变换的理论并没有新的发现，但是对于在计算机系统或者说数字系统中应用离散傅立叶变换，可以说是进了一大步。**快速数论变换(NTT)** 是 **快速傅里叶变换(FFT)** 在数论基础上的实现。

> ## 多项式的表示
>
> ### 点值表示法
>
> $n$  个点可以确定唯一的 $n-1$  次多项式，证明可以考虑高斯消元。
>
> $$
> f(x) = \sum \limits  _{i=0}^{n-1} a_ix^i \iff f(x) = \{(x_0,y_0),(x_1,y_1),\ldots,(x_n,y_n)\}
> $$
>
> ### 系数表示法
>
> 用多项式的各项系数来表示多项式。
>$$f(x) = \sum \limits  _{i=0}^{n-1} a_ix^i \iff f(x) = {a_0,a_1\ldots,a  _{n-1}}$$


将一个多项式由系数表示转化为点值表示，就是 **DFT** 的过程。

将点值表示转化为系数表示，就是 **IDFT** 的过程。

而 **FFT** 就是选取特殊的 $x$  点来加速 **DFT** 与 **IDFT** 的过程。

**FFT** 的经典应用是多项式乘法。

考虑：将两个相乘的多项式转化为点值表示，将点值相乘后再转回系数表示，就完成了多项式乘法。

假定 **DFT** 过程中选取的点相同，有：

$$
\begin{aligned}
& f(x) = \{(x_0,f(x_0)),(x_1,f(x_1)),\ldots,(x_n,f(x_n))\} \\\\
& g(x) = \{(x_0,g(x_0)),(x_1,g(x_1)),\ldots,(x_n,g(x_n))\} \\\\
\end{aligned}
$$

设 $F(x) = f(x) \cdot g(x)$ ，容易得到 $F(x)$  的点值表达：

$$
F(x) = \{(x_0,f(x_0)g(x_0)),(x_1,f(x_1)g(x_1)),\ldots,(x _{n},f(x_n))g(x_n)\}
$$

接下来就需要将点值还原回系数。

如果使用朴素的待定系数高斯消元，对于每个 $ a_i$ 都有 $n$  个方程，依次求解， $ O(n^3) $ 的复杂度不如暴力。

可以"拉格朗日插值" 单点求值，但依然达到了 $ O(n^2)$  的复杂度。

因此我们需要特别地选点。

## 单位复根

![img](https://b3logfile.com/siyuan/1609132319768/assets/20210109110549-l9w25pp.png)

上图为一个单位圆，单位圆上的向量模长均为 $1$ .

根据复数运算法则，两个复数相乘，在复平面上表现为模长相乘，辐角相加。因此两个模长为 $1$  的向量相乘得到的依然是模长为 $1$  的向量，辐角为两向量之和。

假定 $w$  是复平面上一个单位向量， $w^k=1$ ，说明 $w$ 辐角的 $k$  倍是 $2\pi$ ，即恰好将圆周 $k$  等分。

我们将符合以上条件的复数称为复根，用 $w$  表示。

### 定义

形式地，将 $x^n = 1$ 在复数意义下的解称为 $n$  次复根。

这样的解有 $n$  个。

考虑设 $x=(\cos \alpha,\sin \alpha)$ ，要求 $\alpha \times n = k \times 2\pi$ ，其中 $k$  为整数。

移项得到 $\alpha = \dfrac{k}{n} \times 2\pi$ ，显然有 $n$  个解。

那么：

$$
\omega _n ^k = \cos \dfrac{k}{n} \times 2\pi + i \sin \dfrac{k}{n} \times2\pi
$$

将 $w_n^1$ 即 $w_n$ 称为 $n$  次单位复根。

![img](https://b3logfile.com/siyuan/1609132319768/assets/20210109122735-88v0cy7.png)

在上图中， $w_4^1 = i$ 表示 $4$  次单位复根。

可以发现， $n$  次单位复根表示将单位圆 $n$  等分后，从 $x$  轴正方向开始逆时针第一个角所对应的复数。

### 性质

单位复根有三个重要性质：

$$
\begin{aligned}
& \omega _n ^n = 1 \\\\
& \omega _n ^k = \omega _{2n} ^{2k} \\\\
& \omega _{2n} ^{k+n} = -\omega _{2n} ^k \\\\
\end{aligned}
$$

第一个很好理解，由"定义"就能知道。
第二个也很好理解，考虑 $n$ 等分下前 $k$  个角与 $2n$ 等分下前 $2k$  个角等价。

第三个稍微需要证明。

$$
\begin{aligned}
& \omega _{2n}^{k+n} = (\cos (\dfrac{k+n}{2n} \times 2\pi),\sin (\dfrac{k+n}{2n} \times 2\pi))\\\\
& \cos(\dfrac{k+n}{2n} \times 2\pi) \iff \cos (\dfrac{k}{2n} \times 2\pi + \pi) \iff -\cos(\dfrac{k}{2n} \times 2\pi)\\\\
& \sin(\dfrac{k+n}{2n} \times 2\pi) \iff \sin (\dfrac{k}{2n} \times 2\pi + \pi) \iff -\sin(\dfrac{k}{2n} \times 2\pi)\\\\
& \omega _{2n}^{k+n} = (\cos (\dfrac{k+n}{2n} \times 2\pi),\sin (\dfrac{k+n}{2n} \times 2\pi))\\\\
& = (-\cos(\dfrac{k}{2n} \times 2\pi),-sin(\dfrac{k}{2n} \times 2\pi)))\\\\
& = -\omega _{2n}^k \\\\
\end{aligned}
$$

---

单位复根有什么用？在后面的处理中，会频繁地用到其特殊性质。

## 快速傅里叶变换

**FFT** 的核心思想是 **分治**。就 **DFT** 来说，它分治地求解 $x=\omega_n^k$ 时的 $f(x)$  的值。

也就是，它选用单位复根的 $0$ 到 $ n - 1$ 次幂作为点值，进行系数表示转点值表示的操作。

其分治思想体现在将多项式**分为奇次项与偶次项**处理，从而使暴力代入的 $ O(n^2) $  复杂度降为 $ O(n\log n)$ .

以共有 $8$  项的  $7$  次多项式为例：

$$
f(x) = a_0 + a_1x + a_2x^2 + a_3x^3 + a_4x^4 + a_5x^5 + a_6x^6 + a_7x^7
$$

按照次数奇偶分组后，对奇次项组提出一个 $x$ ：

$$
\begin{aligned}
f(x) & = a_0 + a_1x + a_2x^2 + a_3x^3 + a_4x^4 + a_5x^5 + a_6x^6 + a_7x^7 \\\\
& = (a_0 + a_2x^2 + a_4x^4 + a_6x^6) + (a_1x + a_3x^3 + a_5x^5 + a_7x^7) \\\\
& = (a_0 + a_2x^2 + a_4x^4 + a_6x^6) + x(a_1 + a_3x^2 + a_5x^4 + a_7x^6)
\end{aligned}
$$

两边形式高度类似，构造两个新函数：

$$
\begin{align}
F(x) = a_0 + a_2x + a_4x^2 + a_6x^3 \\\\
G(x) = a_1 + a_3x + a_5x^2 + a_7x^3 \\\\
\end{align}
$$

那么：

$$
f(x) = F(x^2) + x \times G(x^2)
$$

接下来用上单位复根的性质：

$$
\begin{aligned}
\operatorname{DFT}(f(\omega _{n}^k)) & = \operatorname{DFT}(F(\omega_n^{2k})) + \omega_n^k \times \operatorname{DFT}(G(\omega_n^{2k})) \\\\
& = \operatorname{DFT}(F(\omega _{n/2}^{k})) + \omega_n^k \times \operatorname{DFT}(G(\omega _{n/2}^{k}))
\end{aligned}
$$

同理有：

$$
\begin{aligned}
\operatorname{DFT}(f(\omega _{n}^{k+n/2})) & = \operatorname{DFT}(F(\omega_n^{2k+n})) + \omega_n^{k+n/2} \times \operatorname{DFT}(G(\omega_n^{2k+n})) \\\\
& = \operatorname{DFT}(F(\omega_n^{2k})) - \omega_n^k \times \operatorname{DFT}(G(\omega_n^{2k})) \\\\
& = \operatorname{DFT}(F(\omega _{n/2}^{k})) - \omega_n^k \times \operatorname{DFT}(G(\omega _{n/2}^{k}))
\end{aligned}
$$

那么只要求出 $\operatorname{DFT}(F(\omega _{n/2}^k))$ 与 $\operatorname{DFT}(G(\omega _{n/2}^k))$ ，就可以求出 $\operatorname{DFT}(f(\omega _{n}^{k}))$ 与 $\operatorname{DFT}(f(\omega _{n}^{k+n/2}))$ .

为什么需要区分左半边和右半边呢？因为 $2k > n$ 时，可以将 $2k$ 减去一个周期 $n$ ，实质上计算的是一样的东西。

递归求解即可。容易发现一层的时间复杂度至多是 $O(n)$ ，一共 $\log n$ 层，总复杂度 $O(n \log n)$ .

在实现中，需要注意分治 DFT 处理的多项式的长度必须是 $2^m,m \in \mathbf{N^*}$ ，所以需要将原多项式补充长度。

---

```cpp
const double PI = 3.14159265358979;
void DFT(comp* f, int n)
{
    if (n == 1) return;
    for (int i = 0; i < n; ++i) tmp[i] = f[i];
    for (int i = 0; i < n; i += 2) f[i >> 1] = tmp[i];               //0,2,4,8,...,n - 2 left
    for (int i = 1; i < n; i += 2) f[(n >> 1) + (i >> 1)] = tmp[i];  //1,3,5,...,n - 1 right
    comp* F = f, *G = f + (n >> 1);
    DFT(F, n >> 1), DFT(G, n >> 1);
    comp now = comp(1, 0), w = comp(cos(2 * PI / n), sin(2 * PI / n));
    for (int i = 0; (i << 1) < n; ++i)
    {
        tmp[i] = F[i] + now * G[i];
        tmp[i + (n >> 1)] = F[i] - now * G[i];
        now = now * w;
    }
    for (int i = 0; i < n; ++i) f[i] = tmp[i];
}
```

## 快速傅里叶逆变换

**傅里叶变换逆变换(IDFT)** 的作用，是将目标多项式的点值形式转化成系数形式。

将单位复根代入多项式，可以得到：

$$
\begin{cases}
& y_0 = a_0 + a_1 + a_2 + \ldots + a_n(x_0 = 1)\\\\
& y_1 = a_0 + a_1\omega_n^1 + a_2\omega_n^2 + \ldots + a_n\omega_n^{n-1}(x_1 = \omega_n^1)\\\\
& y_2 = a_0 + a_1\omega_n^2 + a_2\omega_n^4 + \ldots + a_n\omega_n^{2(n-1)}(x_2 = \omega_n^2)\\\\
& \cdots\\\\
& y _{n-1} = a_0 + a_1\omega_n^{n-1} + a_2\omega_n^{2(n-1)} + \ldots + a_n\omega_n^{(n-1)^2}(x _{n-1} = \omega_n^{n-1})
\end{cases}
$$

考虑写成矩阵的形式：

$$\begin{bmatrix} y_0 \\\\ y_1 \\\\ y_2 \\\\ y_3 \\\\ \vdots\\\\ y _{n-1}\\\\ \end{bmatrix} = \begin{bmatrix} 1 & 1 & 1 & 1 & \cdots & 1 \\\\
1 & \omega_n^1 & \omega_n^2 & \omega_n^3 & \cdots & \omega_n^{n-1} \\\\
1 & \omega_n^2 & \omega_n^4 & \omega_n^6 & \cdots & \omega_n^{2(n-1)} \\\\
1 & \omega_n^3 & \omega_n^6 & \omega_n^9 & \cdots & \omega_n^{3(n-1)} \\\\
\vdots & \vdots & \vdots & \vdots & \ddots & \vdots\\\\
1 & \omega_n^{n-1} & \omega_n^{2(n-1)} & \omega_n^{3(n-1)} & \cdots & \omega_n^{(n-1)^2}\\\\
\end{bmatrix}
\times 
\begin{bmatrix}
a_0\\\\
a_1\\\\
a_2\\\\
a_3\\\\
\vdots\\\\
a _{n-1}\\\\
\end{bmatrix}
$$

即表示为 $A = B \times C$ ，现在已知 $A$ ， $B$  要求 $C$ .

考虑求出 $B$  的逆矩阵 $B'$ ，那么： $C = A \times B'$ .

问题在于如何求 $B'$ ，使用常规的高斯消元法逆矩阵需要 $O(n^3)$  的时间，显然是不可接受的。

考虑直接构造！

### 构造逆矩阵

$$
\begin{bmatrix} 1 & 1 & 1 & 1 & \cdots & 1 \\\\ 1 & \omega_n^1 & \omega_n^2 & \omega_n^3 & \cdots & \omega_n^{n-1} \\\\
1 & \omega_n^2 & \omega_n^4 & \omega_n^6 & \cdots & \omega_n^{2(n-1)} \\\\
1 & \omega_n^3 & \omega_n^6 & \omega_n^9 & \cdots & \omega_n^{3(n-1)} \\\\
\vdots & \vdots & \vdots & \vdots & \ddots & \vdots\\\\
1 & \omega_n^{n-1} & \omega_n^{2(n-1)} & \omega_n^{3(n-1)} & \cdots & \omega_n^{(n-1)^2}\\\\ \end{bmatrix} \times \begin{bmatrix}
\dfrac{1}{n} & \dfrac{1}{n} & \dfrac{1}{n} & \dfrac{1}{n} & \cdots & \dfrac{1}{n} \\\\
\dfrac{1}{n} & \dfrac{1}{n \omega_n^1} & \dfrac{1}{n\omega_n^2} & \dfrac{1}{n\omega_n^3} & \cdots & \dfrac{1}{n\omega_n^{n-1}} \\\\
\dfrac{1}{n} & \dfrac{1}{n\omega_n^2} & \dfrac{1}{n\omega_n^4} & \dfrac{1}{n\omega_n^6} & \cdots & \dfrac{1}{n\omega_n^{2(n-1)}} \\\\
\dfrac{1}{n} & \dfrac{1}{n\omega_n^3} & \dfrac{1}{n\omega_n^6} & \dfrac{1}{n\omega_n^9} & \cdots & \dfrac{1}{n\omega_n^{3(n-1)}} \\\\
\vdots & \vdots & \vdots & \vdots & \ddots & \vdots\\\\
\dfrac{1}{n} & \dfrac{1}{n\omega_n^{n-1}} & \dfrac{1}{n\omega_n^{2(n-1)}} & \dfrac{1}{n\omega_n^{3(n-1)}} & \cdots & \dfrac{1}{n\omega_n^{(n-1)^2}}\\\\
\end{bmatrix} =
\begin{bmatrix}
1 & 0 & 0 & 0 & \cdots & 0 \\\\
0 & 1 & 0 & 0 & \cdots & 0 \\\\
0 & 0 & 1 & 0 & \cdots & 0 \\\\
0 & 0 & 0 & 1 & \cdots & 0 \\\\
\vdots & \vdots & \vdots & \vdots & \ddots & \vdots\\\\
0 & 0 & 0 & 0 & \cdots & 1 \\\\
\end{bmatrix}
$$

**每个位置取倒数后再除以 $n$  即可得到逆矩阵。**

考虑为什么。对于一个位置 $(i,i)$ ，其值为 $\sum \limits  _{j=1}^n a(i,j) \times b(j,i)$ ，容易观察得到这两个矩阵都关于对角线对称，那么 $a(i,j) = a(j,i)$ 后， $b(j,i) = \dfrac{1}{n \times a(j,i)}$ 使每一项 $a(i,j) \times b(j,i) = \dfrac{1}{n}$ ，累加即可得到 $1$ .

稍微证明一下为何关于对角线对称：

考虑 $a(i,j) = (\omega_n^{i-1})^{j-1} = (\omega_n^{j-1})^{i-1} = a(j,i)$ .

证毕。

考虑对于位置 $(i,j) \text{ where } i \neq j$ ，其值等于零。

$$
\begin{align}
 \sum \limits  _{k=1}^n a(i,k) \times b(k,j) & = \dfrac{1}{n} \times \sum \limits  _{k=1} \omega_n^{(i-1)(k-1)} \times \dfrac{1}{\omega_n^{(k-1)(j-1)}} \\\\
& = \dfrac{1}{n}\times \sum \limits  _{k=1} \omega_n^{(i-1)(k-1)} \times \omega_n^{(1-j)(k-1)} \\\\
& = \dfrac{1}{n} \times \sum \limits  _{k=1} \omega_n^{(k-1)(i-j)} \\\\
& = \dfrac{1}{n} \times 0 \\\\
\end{align}
$$

单位圆上转一圈，正负成对消去，最终为零。

---

此时我们得到了逆矩阵，但是要如何使用？

考虑到 **DFT** 的过程实质上是对 $C$ 矩阵即系数矩阵做线性变换得到 $A$ 矩阵，即点值矩阵。

整个 **DFT** 的过程相当于给传入的矩阵乘上了 $B$  矩阵。

现在 **IDFT** 的过程中，我们需要给传入的矩阵乘上 $B'$ 矩阵。由于 $B$  矩阵的每个元素都对应 $B'$ 矩阵上的一个元素，可以用类似 **DFT** 的写法。

单位复根的倒数： $\dfrac{1}{\omega_n^k} \times \omega_n^k = 1$ ，考虑复数乘法的几何意义，即辐角相加，那么 $\dfrac{1}{\omega_n^k}$ 的辐角为 $\omega_n^k$ 辐角的相反数，关于 $y$  轴对称，虚部系数取相反数即可。

$$
\dfrac{1}{\omega_n^k} = \cos (\dfrac{2\pi k}{n}) - i \sin(\dfrac{2\pi k}{n})
$$

---

代码实现只需要在 **DFT** 的基础上改一个符号。甚至可以加一个调节变量合在一起。

```cpp
void IDFT(comp* f, int n)
{
    if (n == 1) return;
    for (int i = 0; i < n; ++i) tmp[i] = f[i];
    for (int i = 0; i < n; i += 2) f[i >> 1] = tmp[i];               //0,2,4,8,...,n - 2 left
    for (int i = 1; i < n; i += 2) f[(n >> 1) + (i >> 1)] = tmp[i];  //1,3,5,...,n - 1 right
    comp* F = f, *G = f + (n >> 1);
    DFT(F, n >> 1), DFT(G, n >> 1);
    comp now = comp(1, 0), w = comp(cos(2 * PI / n), -sin(2 * PI / n));
    for (int i = 0; (i << 1) < n; ++i)
    {
        tmp[i] = F[i] + now * G[i];
        tmp[i + (n >> 1)] = F[i] - now * G[i];
        now = now * w;
    }
    for (int i = 0; i < n; ++i) f[i] = tmp[i];
}
```

```cpp
void DFT(comp* f, int n, int rev = 1)
{
    if (n == 1) return;
    for (int i = 0; i < n; ++i) tmp[i] = f[i];
    for (int i = 0; i < n; i += 2) f[i >> 1] = tmp[i];               //0,2,4,8,...,n - 2 left
    for (int i = 1; i < n; i += 2) f[(n >> 1) + (i >> 1)] = tmp[i];  //1,3,5,...,n - 1 right
    comp* F = f, *G = f + (n >> 1);
    DFT(F, n >> 1), DFT(G, n >> 1);
    comp now = comp(1, 0), w = comp(cos(2 * PI / n), rev * sin(2 * PI / n));
    for (int i = 0; (i << 1) < n; ++i)
    {
        tmp[i] = F[i] + now * G[i];
        tmp[i + (n >> 1)] = F[i] - now * G[i];
        now = now * w;
    }
    for (int i = 0; i < n; ++i) f[i] = tmp[i];
}
```

### 位逆序置换

位逆序置换可以将递归实现改成迭代实现，从而减小常数。

以一个 $8$  次多项式为例：  
初始为 $\{x_0,x_1,x_2,x_3,x_4,x_5,x_6,x_7\}$ .  
第二层为 $\{x_0,x_2,x_4,x_6\}\{x_1,x_3,x_5,x_7\}$ .  
第三层为 $\{x_0,x_4\}\{x_2,x_6\}\{x_1,x_5\}\{x_3,x_7\}$ .  
第四层为 $\{x_0\}\{x_4\}\{x_2\}\{x_6\}\{x_1\}\{x_5\}\{x_3\}\{x_7\}$ .  

规律：原序列中每个数，表示为二进制下，将二进制表示翻转得到最终该位置的下标。例如 $x_1$ 为 $(001)_2$ ，翻转后为 $(100)_2$ ，即 $x_4$ .

常用一种 $O(n)$  的逆位序变换方法。用递推实现。

设 $len = 2^k$ ，其中 $k$  表示二进制长度。

设 $R(x)$ 代表长度为 $k$ 的二进制数 $x$  翻转后的数。

要求 $R(0),R(1),\ldots,R(n-1) $ .

首先 $R(0) = 0$ .

在求 $R(x)$ 时，已知 $R(\lfloor \dfrac{x}{2} \rfloor)$ ，将 $x$  右移一位后翻转，翻转的结果再右移一位，就得到了除了个位外的翻转结果。考虑个位会翻转到首位，如果 $x$ 是奇数，结果加上最高位对应的 $2^{k-1} =\dfrac{len}{2}$ 即可。

$$
R(x) = \lfloor \dfrac{R(\lfloor \dfrac{x}{2} \rfloor)}{2} \rfloor + (x \bmod 2) \times \dfrac{len}{2}
$$

---

代码实现：

```cpp
void change(comp* f, int n)
{
    static int R[maxn], top = 1;
    if (top < n - 1)
    {
    	for (int i = top; i < n; ++i)
        	R[i] = (R[i >> 1] >> 1) | ((i & 1) * (n >> 1));
        top = n - 1;
    }
    for (int i = 0; i < n; ++i)
        if (i < R[i]) std::swap(f[i], f[R[i]]);
}

void FFT(comp* f, int n, int rev)
{
    change(f, n);
    for (int h = 2; h <= n; h <<= 1)  //the len of each block now
    {
        comp w = comp(cos(2 * PI / h), rev * sin(2 * PI / h));  //blocklen determines w step
        for (int i = 0; i < n; i += h)
        {
            comp now = comp(1, 0);  //each small block
            for (int j = i; j < i + (h >> 1); ++j)
            {
                comp F = f[j], G = now * f[j + (h >> 1)];
                f[j] = F + G, f[j + (h >> 1)] = F - G;  //left and right
                now = now * w;
            }
        }
    }
    if (rev == -1)
        for (int i = 0; i < n; ++i) f[i].x /= n;  //IDFT
}
```

## 多项式乘法

多项式乘法是 **FFT** 的经典应用。

> {{{
> **FFT** 的经典应用是多项式乘法。  
> 考虑：将两个相乘的多项式转化为点值表示，将点值相乘后再转回系数表示，就完成了多项式乘法。  
> 假定 **DFT** 过程中选取的点相同，有：  
>
> $$
> \begin{aligned}
> & f(x) = \{(x_0,f(x_0)),(x_1,f(x_1)),\ldots,(x _{n},f(x_n))\} \\\\
> & g(x) = \{(x_0,g(x_0)),(x_1,g(x_1)),\ldots,(x _{n},g(x_n))\} \\\\
> \end{aligned}
> $$
>
> 设 $F(x) = f(x) \cdot g(x)$ ，容易得到 $F(x)$  的点值表达：
>
> $$
> F(x) = \{(x_0,f(x_0)g(x_0)),(x_1,f(x_1)g(x_1)),\ldots,(x _{n},f(x_n))g(x_n)\}
> $$
>
> 接下来就需要将点值还原回系数。
>
> }}}
>

将相乘的 $f(x)$ 与 $g(x)$ 转为点值表示后，将 $y$ 值相乘，再转回系数表示即可。

---

有了前面的铺垫，代码不难实现了。这里给出最朴素的递归 **FFT** 的实现：

```cpp
#include <cstdio>
#include <algorithm>
#include <cmath>
#include <ctype.h>
const int bufSize = 1e6;
//#define DEBUG
inline char nc()
{
    #ifdef DEBUG
    return getchar();
    #endif
    static char buf[bufSize], *p1 = buf, *p2 = buf;
    return p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, bufSize, stdin), p1 == p2) ? EOF : *p1++;
}
template<typename T>
inline T read(T &r)
{
    static char c;
    static int flag;
    flag = 1, r = 0;
    for (c = nc(); !isdigit(c); c = nc()) if (c == '-') flag = -1;
    for (; isdigit(c); c = nc()) r = r * 10 + c - 48;
    return r *= flag;
}
const int maxn = 4e6 + 100;
const double PI = 3.1415926535897932384626433832;
struct comp
{
    double x, y;
    comp(double nx = 0, double ny = 0) { x = nx, y = ny; }
} f[maxn], g[maxn], tmp[maxn];
comp operator+( comp a,  comp b) { return comp(a.x + b.x, a.y + b.y); }
comp operator-( comp a,  comp b) { return comp(a.x - b.x, a.y - b.y); }
comp operator*( comp a,  comp b) { return comp(a.x * b.x - a.y * b.y, a.x * b.y + a.y * b.x); }
void DFT(comp* f, int n, int rev = 1)
{
    if (n == 1) return;
    for (int i = 0; i < n; ++i) tmp[i] = f[i];
    for (int i = 0; i < n; i += 2) f[i >> 1] = tmp[i];               //0,2,4,8,...,n - 2 left
    for (int i = 1; i < n; i += 2) f[(n >> 1) + (i >> 1)] = tmp[i];  //1,3,5,...,n - 1 right
    comp* F = f, *G = f + (n >> 1);
    DFT(F, n >> 1, rev), DFT(G, n >> 1, rev);
    comp now = comp(1, 0), w = comp(cos(2.0 * PI / n), rev * sin(2.0 * PI / n));
    for (int i = 0; i < (n >> 1); ++i)
    {
        tmp[i] = F[i] + now * G[i];
        tmp[i + (n >> 1)] = F[i] - now * G[i];
        now = now * w;
    }
    for (int i = 0; i < n; ++i) f[i] = tmp[i];
}
int n, m;
int main()
{
    read(n), read(m);
    for (int i = 0; i <= n; ++i) read(f[i].x);
    for (int i = 0; i <= m; ++i) read(g[i].x);
    int limit = 1;
    while (limit <= n + m) limit <<= 1;
    DFT(f, limit, 1), DFT(g, limit, 1);
    for (int i = 0; i <= limit; ++i) f[i] = f[i] * g[i];
    DFT(f, limit, -1);
    for (int i = 0; i <= n + m; ++i) printf("%d ", (int)(f[i].x / limit + 0.5));
    return 0;
}
```

---

### 三次变两次优化

还有一种黑科技，可以将卷积需要的三次 FFT 降低到两次。

考虑 $f(x) \cdot g(x)$ ，将 $g(x)$ 放到 $f(x)$ 的虚部上，求出 $f(x)^2$ 后，虚部除以二就是答案。

$$
(a + bi)^2 = (a^2-b^2) + (2abi)
$$

这样就可以降低不少常数。

### 预处理单位根

此外，还可以预处理 $\omega$ 数组来避免重复计算。

---

完整的板子：

```cpp
#include <cstdio>
#include <algorithm>
#include <cmath>
#include <ctype.h>
const int bufSize = 1e6;
//#define DEBUG
inline char nc()
{
    #ifdef DEBUG
    return getchar();
    #endif
    static char buf[bufSize], *p1 = buf, *p2 = buf;
    return p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, bufSize, stdin), p1 == p2) ? EOF : *p1++;
}
template<typename T>
inline T read(T &r)
{
    static char c;
    static int flag;
    flag = 1, r = 0;
    for (c = nc(); !isdigit(c); c = nc()) if (c == '-') flag = -1;
    for (; isdigit(c); c = nc()) r = r * 10 + c - 48;
    return r *= flag;
}
const int maxn = 3e6 + 100;
const double PI = 3.1415926535897932384626433832;
struct comp
{
    double x, y;
    comp(double nx = 0, double ny = 0) { x = nx, y = ny; }
} f[maxn], tmp[maxn];
comp operator+(const comp& a, const comp& b) { return comp(a.x + b.x, a.y + b.y); }
comp operator-(const comp& a, const comp& b) { return comp(a.x - b.x, a.y - b.y); }
comp operator*(const comp& a, const comp& b) { return comp(a.x * b.x - a.y * b.y, a.x * b.y + a.y * b.x); }
void change(comp* f, int n)
{
    static int R[maxn], top = 1;
    if (top < n - 1)
    {
    	for (int i = top; i < n; ++i)
        	R[i] = (R[i >> 1] >> 1) | ((i & 1) * (n >> 1));
        top = n - 1;
    }
    for (int i = 0; i < n; ++i)
        if (i < R[i]) std::swap(f[i], f[R[i]]);
}

void FFT(comp* f, int n, int rev)
{
    change(f, n);
    for (int h = 2; h <= n; h <<= 1)  //the len of each block now
    {
        comp w = comp(cos(2 * PI / h), rev * sin(2 * PI / h));  //blocklen determines w step
        tmp[0] = comp(1, 0);
        for (int i = 1; i < (h >> 1); ++i) tmp[i] = tmp[i - 1] * w;
        for (int i = 0; i < n; i += h)
        {
            for (int j = i; j < i + (h >> 1); ++j)
            {
                comp F = f[j], G = tmp[j - i] * f[j + (h >> 1)];
                f[j] = F + G, f[j + (h >> 1)] = F - G;  //left and right
            }
        }
    }
}
int n, m;
int main()
{
    read(n), read(m);
    for (int i = 0; i <= n; ++i) read(f[i].x);
    for (int i = 0; i <= m; ++i) read(f[i].y);
    int limit = 1;
    while (limit <= n + m) limit <<= 1;
    FFT(f, limit, 1);
    for (int i = 0; i <= limit; ++i) f[i] = f[i] * f[i];
    FFT(f, limit, -1);
    for (int i = 0; i <= n + m; ++i) printf("%d ", (int)((f[i].y / 2 / limit) + 0.5));
    return 0;
}
```

### 高精度乘法

可以利用 FFT 来加速高精度乘法。

具体地，将高精度整数写成 $\sum \limits _{i=0}^n a_i{10}^i$ ，就转化为了多项式，然后利用 FFT 解决多项式乘法，转换回高精度整数后处理进位即可。

有时可能会有精度上的问题。

---

代码实现：

```cpp
#include <cstdio>
#include <algorithm>
#include <cstring>
#include <cmath>
#include <ctype.h>
const int bufSize = 1e6;
inline char nc()
{
    #ifdef DEBUG
    return getchar();
    #endif
    static char buf[bufSize], *p1 = buf, *p2 = buf;
    return p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, bufSize, stdin), p1 == p2) ? EOF : *p1++;
}
template<typename T>
inline T read(T &r)
{
    static char c;
    static int flag;
    flag = 1, r = 0;
    for (c = nc(); !isdigit(c); c = nc()) if (c == '-') flag = -1;
    for (; isdigit(c); c = nc()) r = r * 10 + c - 48;
    return r *= flag;
}
inline void read(char *s)
{
    static char c;
    for (c = nc(); !isdigit(c); c = nc());
    for (; isdigit(c); c = nc()) *s++ = c;
    *s = '\0';
}
const int maxn = 3e6 + 100;
struct comp
{
    double x, y;
    comp(const double nx = 0, const double ny = 0) { x = nx, y = ny; }
} f[maxn], tmp[maxn];
comp operator+(const comp& a, const comp& b) { return comp(a.x + b.x, a.y + b.y); }
comp operator-(const comp& a, const comp& b) { return comp(a.x - b.x, a.y - b.y); }
comp operator*(const comp& a, const comp& b) { return comp(a.x * b.x - a.y * b.y, a.x * b.y + a.y * b.x); }
int R[maxn];
const double PI = 3.141592653589793238462;
void change(comp* f, int n)
{
    static int R[maxn], top = 1;
    if (top < n - 1)
    {
    	for (int i = top; i < n; ++i)
        	R[i] = (R[i >> 1] >> 1) | ((i & 1) * (n >> 1));
        top = n - 1;
    }
    for (int i = 0; i < n; ++i)
        if (i < R[i]) std::swap(f[i], f[R[i]]);
}
void DFT(comp* f, int n, int rev)
{
    change(f, n);
    for (int h = 2; h <= n; h <<= 1)
    {
        comp w = comp(cos(2.0 * PI / h), rev * sin(2.0 * PI / h));
        tmp[0] = comp(1, 0);
        for (int i = 1; i < (h >> 1); ++i) tmp[i] = tmp[i - 1] * w;
        for (int i = 0; i < n; i += h)
            for (int j = i; j < i + (h >> 1); ++j)
            {
                comp F = f[j], G = tmp[j - i] * f[j + (h >> 1)];
                f[j] = F + G, f[j + (h >> 1)] = F - G;
            }
    }
}
int n, m;
char A[maxn], B[maxn];
int C[maxn];
int main()
{
    read(A), read(B), n = strlen(A), m = strlen(B);
    for (int i = 0; i < n; ++i) f[i].x = A[n - i - 1] - '0';
    for (int i = 0; i < m; ++i) f[i].y = B[m - i - 1] - '0';
    int limit = 1;
    while (limit < n + m - 1) limit <<= 1;
    DFT(f, limit, 1);
    for (int i = 0; i <= limit; ++i) f[i] = f[i] * f[i];
    DFT(f, limit, -1);
    for (int i = 0; i <= limit; ++i) C[i] = (int)(f[i].y / 2 / limit + 0.5);
    for(int i = 0; i <= limit; ++i) if (C[i] > 9) C[i + 1] += C[i] / 10, C[i] %= 10; 
    while (C[limit] > 9) C[limit + 1] += C[limit] / 10, C[limit] %= 10, ++limit;
    while(limit > 0 && C[limit] == 0) --limit;
    for (int i = limit; i >= 0; --i) putchar(C[i] + '0');
    return 0;
}
```