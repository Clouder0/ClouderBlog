---
title: "拓展欧几里德算法(exgcd)学习笔记"
date: 2020-05-23T13:38:12+08:00
slug: "exgcd"
type: post
tags:
  - math
  - exgcd
categories:
  - Algorithm
---

# 拓展欧几里得算法

解不定方程 $ax + by = c$ ，可以使用拓展欧几里得算法。

首先解 $ax + by = \gcd (a,b)$ .

## 欧几里得算法

证明 $\gcd(a,b) = \gcd(b,a \bmod b)$ ：

设 $a = g \times k_1$ ， $b = g \times k_2$ ，其中 $k_1,k_2$ 互质。

要证明 $\gcd(a,b) = \gcd(b,a\bmod b)$ ，即证 $g = \gcd(g \times k_2, (g \times k_1 )\bmod (g \times k_2))$ .

将 $g$ 消去，即证 $\gcd(k_2,k_1 \bmod k_2) = 1$ .

假设有公因子 $x$ ，则：

$k_2 = x \times n_1$ ， $k_1 \bmod k_2 = x \times n_2$ .

可以写出： $k_1 = k \times k_2 + k_1 \bmod k_2$ ，其中 $k$ 为整数。

化简得出： $k_1 = k \times (x \times n_1) + x \times n_2$ ，则 $k_1,k_2$ 有公因子 $x$ ，而 $k_1,k_2$ 应当互质，矛盾。

所以 $\gcd(k_2,k_1 \bmod k_2) = 1$ .

则 $\gcd(a,b) = \gcd(b,a\bmod b)$ .

## 拓展欧几里得算法

证明 $ax + by = \gcd(a,b)$ 一定有解。 $(a,b \neq 0)$ .

由上文结论，该式等价于：

$bx + (a \bmod b)y = \gcd(b,a \bmod b)$ .

辗转相除，当 $b = 0$ 时，方程为： $ax = \gcd(a,b) = a$ ，即用欧几里得算法求最大公因数的最后一步。

有 $x = 1,y = 0$ .

向上还原，即可获得一组解。

---

拓展欧几里得算法用于求出 $ax + by = \gcd(a,b)$ 的一组特解：

$ax_1 + by_1 = \gcd(a,b)$ 可转化为：

$bx_2 + (a \bmod b)y_2 = \gcd(b,a \bmod b)$ .

其中： $\gcd(a,b) = \gcd(b,a \bmod b)$ .

可以得到： $ax_1 + by_1 = bx_2 + (a \bmod b)y_2$ .

即： $ax_1 + by_1 = bx_2 + (a - \lfloor \dfrac{a}{b} \rfloor \times b)y_2$ .

展开、合并同类项得到： $ax_1 + by_1 = ay_2 + b(x_2 - y_2 \times \lfloor \dfrac{a}{b} \rfloor)$

即：

$x_1 = y_2$ ， $y_1 = x_2 - y_2 \times \lfloor \dfrac{a}{b} \rfloor = x_2 - x_1 \times \lfloor \dfrac{a}{b} \rfloor$ .

可以据此写出拓展欧几里得算法。可以使用引用传值的技巧简化代码。

```cpp
int exgcd(int a,int b,int &x,int &y)
{
    if(b == 0) 
    {
        x = 1,y = 0;
        return a;
    }
    int g = exgcd(b, a % b, y, x);
    y -= x * (a / b);
    return g;
}
```

---

此时回归到解 $ax + by = c$ 的问题上来。若 $\gcd(a,b) \mid c$ ，则原方程有整数解。

$ax' + by' = \gcd(a,b)$ ，则：

$$
x = x' \times \dfrac{c}{\gcd(a,b)}
$$

$$
y = y' \times \dfrac{c}{\gcd(a,b)}
$$

若要得到任意组解呢？

记求出的特解为 $x_0,y_0$ .

$ax + by = ax_0 + by_0$ .

$a(x - x_0) = b(y_0-y)$ .

$\dfrac{a}{b} \times (x - x_0) = y_0 - y$ .

$x - x_0$ 需要为整数，则 $\dfrac{b}{\gcd(a,b)} \mid (x' - x_0')$ ，设为 $x' - x_0' = \dfrac{b}{\gcd(a,b)} \times t$ .

可以写出：

$x' = x_0' + \dfrac{b}{\gcd(a,b)} \times t$

$y' = y_0' - \dfrac{a}{\gcd(a,b)} \times t$

由此即可求出任意解，再带回原式即可。

例如求 $x'$ 的最小正整数解，可以发现 $x'$ 任意加减 $b$ 都是合法解，因此可以直接 $((x \bmod b) + b) \bmod b$ .

---

拓展欧几里得算法还可以用于求解线性同余方程。

形如 $ax \equiv c \pmod{b}$ .

可以写成 $ax + by = c$ ，随后按二元一次不定方程的方法求解即可。

---

例题：[青蛙的约会](https://www.luogu.com.cn/problem/P1516)

两只青蛙初始坐标为 $x,y$ ，每次跳 $m,n$ 米，定义相遇为模 $L$ 意义下坐标相等，求一起跳跃多少次后相遇。

形式地， $x + m \times t = y + n \times t \pmod L$ .

$$
x + m \times t + L \times k = y + n \times t
$$

$$
(m-n) \times t + L \times k = y - x
$$

求 $t$ 的最小正整数解。

套上板子即可。

```cpp
#include <algorithm>
#include <cstdio>
#define ll long long
ll sx, sy, m, n, L;
inline ll gcd(ll a, ll b) { return b == 0 ? a : gcd(b, a % b); }
inline void exgcd(ll a, ll b, ll& x, ll& y)
{
    if (b == 0) return (void)(x = 1, y = 0);
    exgcd(b, a % b, y, x), y -= (a / b) * x;
}
int main()
{
    scanf("%lld %lld %lld %lld %lld", &sx, &sy, &m, &n, &L);
    if(m - n < 0) std::swap(m,n),std::swap(sx,sy);
    ll g = gcd(m - n, L);
    ll a = m - n, b = L, x, y, c = sy - sx;
    if (c % g) { puts("Impossible"); return 0; }
    exgcd(a, b, x, y);
    x *= c / g, y *= c / g;
    ll mod = b / g;
    x = ((x % mod) + mod) % mod;
    printf("%lld\n", x);
    return 0;
}
```