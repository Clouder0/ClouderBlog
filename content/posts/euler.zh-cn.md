---
title: "欧拉函数、欧拉定理学习笔记"
date: 2020-06-02T12:55:27+08:00
slug: "euler"
type: posts
tags:
  - math
  - euler
categories:
  - algorithm
---

## 欧拉函数

欧拉函数， $\varphi(n)$ ， $\leq n$ 的与 $n$ 互质的数的个数。

$\varphi(n) = \sum \limits _{i=1}^n \left[ i \nmid n \right]$

例如， $\varphi(1) = 1$ ，而对于质数 $p$ ， $\varphi(p) = p - 1$ .

### 引理 1

若 $p$ 为质数， $\varphi(p^k) = p^k - p^{k-1}$

证明：

$\forall i \leq p^k$ ，有 $\dfrac{p^k}{p}$ 个数满足 $x \mid p^k$ 。

形如： $1,2,3,\ldots,p,p+1,p+2,\ldots,2p,\ldots,p^k$

因此 $\varphi(p^k) = p^k - \dfrac{p^k}{p} = p^k - p^{k-1}$

---

### 与唯一分解定理关系

由整数唯一分解定理，整数 $n$ 可以写作：

$n = \prod \limits _{i=1}^s p_i^{k_i}$ ，其中 $p_i$ 为质数。

有： $\varphi(n) = n \times \prod \limits _{i=1}^{s} \dfrac{p_i - 1}{p_i}$

证明：

已知欧拉函数是积性函数。

$\varphi(p^k) = p^k - p^{k-1} = p^k \times (1 - \dfrac{1}{p})$

$\varphi(n) = \prod \limits _{i=1}^s \varphi(p_i^{k_i})$

$\varphi(n) = \prod \limits _{i=1}^s p_i^{k_i} \times (1 - \dfrac{1}{p_i})$

$\varphi(n) = \prod \limits _{i=1}^s p_i^{k_i} \times \prod \limits _{i=1}^s (1 - \dfrac{1}{p_i})$

$\varphi(n) = n \times \prod \limits _{i=1}^s (1 - \dfrac{1}{p_i})$

由此可以**求单个欧拉函数**。

### 积性函数

积性函数： $\forall a,b,\gcd(a,b) = 1$ ， $\varphi(a\times b) = \varphi(a) \times \varphi(b)$

证明：

可以利用中国剩余定理证明欧拉函数的积性。

$\forall 0 \leq x < a,0 \leq y < b$ ，线性同余方程组：

$\begin{cases} z \equiv x \pmod{a} \\ z \equiv y \pmod{b} \end{cases}$

$z$ 在 $\bmod ab$ 意义下有唯一解。且每个 $z$ 对应的 $x,y$ 是固定的。

当且仅当 $x$ 与 $a$ 互质且 $y$ 与 $b$ 互质时， $z$ 与 $ab$ 互质。

---

否则：

当 $x$ 与 $a$ 不互质时，设 $x = g \times k_1,a = g \times k_2$ ， $z = x + a \times k$ .

$z = g \times k_1 + g \times k_2 \times k$ ，不与 $ab$ 互质。

必要性证毕，接下来证明充分性。

---

$z = x + ak_1$

$z = y + bk_2$

$\gcd(a,x) = \gcd(b,y) = 1$

由欧几里得算法知， $\gcd(x + ak_1,a) = \gcd(a,(x+ak_1) \bmod a)$

即 $\gcd(z,a) = \gcd(a,x) = 1$

同理 $\gcd(z,b) = 1$

则 $z$ 与 $a,b$ 无公因子，而与 $ab$ 也无公因子。

$\gcd(z,ab) = 1$

---

$\forall x,y,\gcd(x,a) = 1,\gcd(y,b) = 1$ ， $\gcd(z,ab) = 1$

那么每个与 $a$ 互质的数，任意与一个与 $b$ 互质的数搭配，即可产生唯一的与 $ab$ 互质的数。

根据乘法原理， $\varphi(ab) = \varphi(a) \times \varphi(b)$ .

## 欧拉定理

### 费马小定理

若 $p$ 为素数， $\gcd(a,p) = 1$ ，则 $a^{p-1} \equiv 1 \pmod{p}$ .

对于任意整数 $a$ ，有 $a^p \equiv a \pmod{p}$ .

### 欧拉定理

若 $\gcd(a,m) = 1$ ， $a^{\varphi(m)} \equiv 1 \pmod{m}$

费马小定理是欧拉定理的一种特殊情况，当 $m$ 是素数时， $\varphi(m) = m - 1$ ， $a^{m-1} \equiv 1 \pmod{p}$ .

### 扩展欧拉定理

$$
a^b \equiv \begin{cases} a^{b \bmod \varphi(p)}, \gcd(a,p) = 1 \\ a^b,\gcd(a,p) \neq 1,b < \varphi(p)  \\ a^{b \bmod \varphi(p) + \varphi(p)},\gcd(a,p) \neq 1,b \geq \varphi(p) \end{cases} \pmod{p}
$$

在使用时，一般简化为：

$$
a^b \equiv \begin{cases} a^{b \bmod \varphi(p)},b < \varphi(p) \\ a^{b \bmod \varphi(p) + \varphi(p)},b \geq \varphi(p) \end{cases} \pmod{p}
$$

只有 $\gcd(a,p) = 1,b \geq \varphi(p)$ 的情况有所特殊，现证明简化无误：

$\because \gcd(a,p) = 1,a^\varphi(p) \equiv 1 \pmod{p}$ ，

$\therefore a^{b \bmod \varphi(p) + \varphi(p)} \equiv a^{b \bmod \varphi(p)} \pmod{p}$

## 线性筛欧拉函数

欧拉筛可以用于筛积性函数。

对于枚举的每个 $i$ ：

若 $i$ 为素数，则 $\varphi(i) = i - 1$ .

随后用 $i$ 与素数表筛合数。

设 $x = i \times j$ ，其中 $j$ 为素数。

每个 $x$ 都只会被筛到一次，而 $j$ 是它的最小质因子。

$i \bmod j = 0$ 时， $i$ 包含了 $x$ 的所有质因子。

$\varphi(x) = n \times \prod \limits _{i=1}^s (1 - \dfrac{1}{p_i})$

$\varphi(x) = j \times i \times \prod \limits _{i=1}^s (1 - \dfrac{1}{p_i})$

$\varphi(x) = j \times \varphi(i)$

而当 $i \bmod j \neq 0$ 时， $i$ 与质数 $j$ 互质，根据欧拉函数积性有： $\varphi(x) = \varphi(i) \times \varphi(j) = \varphi(i) \times (j-1)$

将计算欧拉函数的式子塞到线性筛素数的框架中。

```cpp
#include <cstdio>
const int maxn = 1e7 + 100;
int n, p;
bool notprime[maxn];
int primes[maxn], cnt, phi[maxn];
inline void Euler(int top)
{
    phi[1] = 1, notprime[1] = 1;
    for (int i = 2; i <= top; ++i)
    {
        if (!notprime[i]) primes[++cnt] = i, phi[i] = i - 1;
        for (int j = 1; j <= cnt && i * primes[j] <= top; ++j)
        {
            notprime[i * primes[j]] = 1;
            if ((i % primes[j]) == 0) { phi[i * primes[j]] = phi[i] * primes[j]; break; }
            else phi[i * primes[j]] = phi[i] * (primes[j] - 1);
        }
    }
}
int main()
{
    scanf("%d %d", &n, &p);
    Euler(n);
    int ans = 0;
    for (int i = 1; i <= n; ++i) ans = (ans + 1ll * (phi[i] + i) * (phi[i] + i)) % p;
    printf("%d\n", ans);
    return 0;
}
```

---

例题：[SDOI2008]仪仗队

给定 $n \times n$ 的方阵，从 $(0,0)$ 到 $(n-1,n-1)$ ，问从 $(0,0)$ 到任意非 $(0,0)$ 点，共有多少条本质不同的直线。

可以发现图像关于 $y=x$ 对称，可以考虑下半部分的贡献。

本质相同的直线是什么？

设 $(0,0)$ 到 $(x_1,y_1)$ 与 $(x_2,y_2)$ 的直线本质相同，有：

$\dfrac{y_1}{x_1} = \dfrac{y_2}{x_2}$

$\dfrac{x_2}{x_1} = \dfrac{y_2}{y_1} = t$

那么 $x_2 = x_1 \times t,y_2 = y_1 \times t$

当 $(x_1,y_1)$ 是该直线上离原点最近的点时， $t$ 为整数。

并且 $\gcd(x_1,y_1) = 1$ ，否则设 $\gcd(x_1,y_1) = g$ ，点 $(\dfrac{x_1}{g},\dfrac{x_2}{g})$ 一定离原点更近。

而对于后方所有点， $\gcd(x_2,y_2) = t$ ，都不计贡献。

单独考虑 $x = 0,y = 0,y = x$ 的情况。

所求即：

$$
3 + 2 \times \sum \limits _{x=1}^{n-1} \sum \limits _{y=1}^{x-1} [\gcd(x,y) = 1]
$$

可以发现， $\sum \limits _{y=1}^{x-1} [\gcd(x,y) = 1]$ 即欧拉函数， $\varphi(x)$ .

然而，特殊地， $x = 1$ 时， $\varphi(x) = 1$ ，而应当只有 $0$ ，特殊设置一下即可。

原式化简为 $3 + 2 \times \sum \limits _{x=2}^{n-1} \varphi(x)$

特别地，当 $n = 1$ 时，答案为 $0$ .

```cpp
#include <cstdio>
const int maxn = 4e4 + 100;
bool notprime[maxn];
int primes[maxn], cnt, phi[maxn];
int n;
inline void euler()
{
    for (int i = 2; i <= n - 1; ++i)
    {
        if (!notprime[i]) primes[++cnt] = i, phi[i] = i - 1;
        for (int j = 1; j <= cnt && primes[j] * i <= n - 1; ++j)
        {
            notprime[i * primes[j]] = 1;
            if ((i % primes[j]) == 0) { phi[i * primes[j]] = phi[i] * primes[j]; break; }
            phi[i * primes[j]] = phi[i] * (primes[j] - 1);
        }
    }
}
int main()
{
    scanf("%d", &n);
    euler();
    long long ans = 0;
    for (int i = 2; i < n; ++i) ans += phi[i];
    ans = ans * 2 + 3;
    if(n == 1) ans = 0;
    printf("%lld\n",ans);
    return 0;
}
```

---

例题：洛谷 P5091 扩展欧拉函数模板

求  $a^b \pmod{m}$ ，其中 $p \leq 10^{2 \times 10^7}$ .

可以使用扩展欧拉函数减小指数，以简化计算。

先用质因数分解的方法求出 $\varphi(m)$ ，时间复杂度 $O(\sqrt{m})$ .

随后边读边模，最后用快速幂求解即可。

```cpp
#include <cmath>
#include <cstdio>
#include <ctype.h>
#define int long long
const int bufSize = 1e6;
inline char nc()
{
    static char buf[bufSize], *p1 = buf, *p2 = buf;
    return p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, bufSize, stdin), p1 == p2) ? EOF : *p1++;
}
template <typename T>
inline T read(T& r)
{
    static char c;
    r = 0;
    for (c = nc(); !isdigit(c); c = nc()) ;
    for (; isdigit(c); c = nc()) r = r * 10 + c - 48;
    return r;
}
int a, m, b;
inline int euler(int x)
{
    int top = std::sqrt(x);
    int res = x;
    for (int i = 2; i <= top; ++i)
    {
        if ((x % i) == 0)
        {
            res = res / i * (i - 1);
            while ((x % i) == 0) x /= i;
        }
    }
    if (x != 1) res = res / x * (x - 1);
    return res;
}
inline int mul(int x, int y)
{
    int res = 0;
    for (; y > 0; y >>= 1)
    {
        if (y & 1) res = (res + x) % m;
        x = (x + x) % m;
    }
    if (res < 0) res += m;
    return res;
}
inline int fastpow(int x, int k)
{
    int res = 1;
    for (; k; k >>= 1)
    {
        if (k & 1) res = mul(res, x);
        x = mul(x, x);
    }
    return res;
}
signed main()
{
    read(a), read(m);
    int phi = euler(m);
    static char c;
    b = 0;
    for (c = nc(); !isdigit(c); c = nc());
    bool flag = 0;
    for (; isdigit(c); c = nc())
    {
        b = b * 10 + c - 48;
        if (b >= phi) b %= phi, flag = 1;
    }
    printf("%lld\n", fastpow(a, b + phi * flag));
    return 0;
}
```