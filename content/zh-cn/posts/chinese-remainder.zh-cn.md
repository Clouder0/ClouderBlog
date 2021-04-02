---
title: "中国剩余定理学习笔记"
date: 2020-06-12T12:56:40+08:00
slug: "chinese-remainder"
type: posts
tags:
  - chinese-remainder
  - math
categories:
  - Algorithm
---


## 中国剩余定理

$$
\begin{cases} x \equiv a_1 \pmod {p_1} \\ x \equiv a_2 \pmod {p_2} \\ \ldots \\ x \equiv a_i \pmod {p_i} \end{cases}
$$

中国剩余定理用于求解模数两两互质的线性同余方程组。

对于这种 $x \equiv a_i \pmod{p_i}$ 的线性同余方程组，在 $\bmod \prod \limits _{i=1}^n p_i$ 的意义下有唯一解。

---

设 $M = \prod \limits _{i=1}^n p_i$ ，而 $M_i = \dfrac{M}{p_i}$ .

则解 $x = \sum \limits _{i=1}^{n} M_i \times K_i$ ，其中 $K_i$ 满足：

$$
M_i \times K_i \equiv a_i \pmod{p_i}
$$

对于任意的 $M_i \times K_i$ ，由于 $M_i$ 是所有剩余模数的积，则 $M_i \times K_i \equiv 0 \pmod{p_j} (j \neq i)$ ，因此在 $x$ 中加上 $M_i \times K_i$ 对满足 $x \equiv a_j \pmod{p_j} (j \neq i)$ 无影响。

而每项 $M_i \times K_i$ 都能保证第 $i$ 个线性同余方程得到满足。

只需要解出 $K_i$ 即可。

$$
M_i \times K_i + p_i \times t = a_i
$$

利用扩展欧几里得定理求出 $K_i$ .

由于模数两两互素， $\gcd(M_i,p_i) = 1$ ，即解：

$M_i \times k_i + p_i \times t = 1$ 的 $k_i$ ，然后 $K_i = a_i \times k_i$ .

可以发现， $k_i$ 就是 $M_i$ 在 $\bmod p_i$ 意义下的逆元。

那么就可以得到一个解： $x = \sum \limits _{i=1}^n M_i \times k_i \times a_i$

---

而通解为： $x = k \times M + \sum \limits _{i=1}^n M_i \times k_i \times a_i$

证明：设两个解为 $x_1,x_2$ ，有：

$$
\forall i \in \{1,2,\ldots,n\},x_1 - x_2 \equiv 0 \pmod{p_i}
$$

而 $p_i$ 两两互素，则 $M \mid x_1 - x_2$ ，那么 $x_1 - x_2 = k \times M$ ，则在 $\bmod M$ 意义下有唯一解。

---

```cpp
inline ll CRT()
{
    ll prod = 1,ans = 0;
    for (int i = 1; i <= n; ++i) prod *= a[i];
    for (int i = 1; i <= n; ++i)
    {
        ll x,y;
        exgcd(prod / a[i], a[i], x, y);
        ans = (ans + prod / a[i] * b[i] * x) % prod;
    }
    return (ans + prod) % prod;
}
```

---

例题：[TJOI2009]猜数字

给出 $a_i,b_i$ ，要求最小的 $n$ 使得 $b_i \mid (n - a_i)$ .

改写为 $n - a_i \equiv 0 \pmod {b_i}$ .

即 $n \equiv a_i \pmod{b_i}$

求 $n$ 的最小整数解。

这题会爆 long long，需要用到龟速乘的技巧。

```cpp
#include <cstdio>
#define ll long long
const int maxk = 20;
ll k, a[maxk], b[maxk];
void exgcd(ll a, ll b, ll& x, ll& y)
{
    if (b == 0) return (void)(x = 1, y = 0);
    exgcd(b, a % b, y, x), y -= (a / b) * x;
}
ll mul(ll a,ll b,ll mod)
{
    ll res = 0;
    while(a)
    {
        if(a & 1) res = (res + b) % mod;
        b = (b + b) % mod;
        a >>= 1;
    }
    return res % mod;
}
ll crt()
{
    ll prod = 1, ans = 0;
    for (int i = 1; i <= k; ++i) prod *= b[i];
    for (int i = 1; i <= k; ++i)
    {
        ll x, y, mi = prod / b[i];
        exgcd(mi, b[i], x, y);
        x = ((x % b[i]) + b[i]) % b[i];
        ans = (ans + mul(mul(mi, x, prod), a[i], prod)) % prod;
    }
    return (ans + prod) % prod;
}
int main()
{
    scanf("%lld",&k);
    for (int i = 1; i <= k; ++i) scanf("%lld", a + i);
    for (int i = 1; i <= k; ++i) scanf("%lld", b + i);
    printf("%lld\n", crt());
    return 0;
}
```

## 拓展中国剩余定理

然而如果不保证模数互质，就需要使用拓展中国剩余定理。

模数不互质的话，之前的巧妙构造行不通了。尝试更平凡的列式子考虑：

$$
\begin{cases}x \equiv r_1 \pmod{p_1} \\ x \equiv r_2 \pmod{p_2}\end{cases}
$$

可以写成：

$$
\begin{cases} x = r_1 + p_1 \times k_1 \\ x = r_2 + p_2 \times k_2 \end{cases}
$$

即：

$r_1 + p_1 \times k_1 = r_2 + p_2 \times k_2$

移项得到：

$p_1 \times k_1 - p_2 \times k_2 = r_2 - r_1$

等价于解 $a = p_1$ ， $b = -p_2$ ， $c = r_2 - r_1$ 的 $ax + by = c$ 二元一次不定方程。

可以用拓展欧几里得定理解出可行的 $k_1,k_2$ .

解出一组特解 $k_1',k_2'$ 后，利用前文知识，求出通解：

记 $d=\gcd(p_1,-p_2)$

$$
\begin{cases} k_1 = k_1' + t \times \dfrac{-p_2}{d}\\ k_2 = k_2' - t \times \dfrac{p_1}{d}\end{cases}
$$

$x = r_1 + p_1 \times k_1$ 可以写为：

$x = r_1 + p_1 \times k_1' + p_1 \times t \times \dfrac{-p_2}{d}$

其中只有 $t$ 是自变量，该式子等价于：

$x \equiv r_1 + p_1 \times k_1' \pmod{|p_1 \times \dfrac{-p_2}{d}|}$

其中 $|p_1 \times \dfrac{-p_2}{d}| = \operatorname{lcm}(p_1,p_2)$

通过一系列的变换，我们成功将两个线性同余方程合并为一个线性同余方程。

依次类推，即可求出最后的一个线性同余方程，随后求出原线性同余方程组的解。

在计算过程中注意取模，将可取模的都取模以避免溢出。

---

```cpp
#include <cstdio>
#define ll long long
const int maxn = 1e6 + 100;
int n;
ll p[maxn], r[maxn];
ll exgcd(ll a, ll b, ll& x, ll& y)
{
    if (b == 0)
    {
        x = 1, y = 0;
        return a;
    }
    ll g = exgcd(b, a % b, y, x);
    y -= (a / b) * x;
    return g;
}
ll mul(ll a, ll b, ll mod)
{
    ll res = 0;
    while (b > 0)
    {
        if (b & 1) res = (res + a) % mod;
        a = (a + a) % mod;
        b >>= 1;
    }
    return ((res % mod) + mod) % mod;
}
inline ll exCRT()
{
    ll lcm = p[1], lastr = r[1], k1, k2;
    for (int i = 2; i <= n; ++i)
    {
        ll a = lcm, b = p[i], c = (((r[i] - lastr) % p[i]) + p[i]) % p[i];
        ll g = exgcd(a, b, k1, k2);
        ll mod = p[i] / g;
        k1 = ((k1 % mod) + mod) % mod;
        k1 = mul(k1, c / g, mod);

        lastr += k1 * lcm;
        lcm = lcm / g * p[i];
        lastr = ((lastr % lcm) + lcm) % lcm;
    }
    return lastr;
}
int main()
{
    scanf("%d", &n);
    for (int i = 1; i <= n; ++i) scanf("%lld %lld", p + i, r + i);
    printf("%lld\n", exCRT());
    return 0;
}
```