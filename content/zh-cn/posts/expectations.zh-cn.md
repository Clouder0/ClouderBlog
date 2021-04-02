---
title: "[笔记]概率与期望"
date: 2020-08-20T16:03:11+08:00
slug: "expectations"
type: posts
tags:
  - math
  - expectation
  - dynamic-programming
categories:
  - Algorithm
---

## 前言

今天学习一下期望 DP，写点笔记。
由于概率与期望是高中数学内容，已经学过了，不再过多描述。

## 模型

写转移方程时算上概率罢了，常常用逆推。

## 例题

学习知识点的最好方法就是刷题。

### SPOJ Favorite Dice

给一个 $n$ 面的骰子，问每面都被抛到的期望次数。

设 $f(i)$ 为剩余 $i$ 面要被抛到的期望次数。
$f(n) = f(n-1) + 1$，第一次抛怎么抛都可以。
$f(n-1) = \dfrac{n-1}{n} \times f(n-2) + \dfrac{1}{n} \times f(n-1) + 1$
$f(n-2) = \dfrac{n-2}{n} \times f(n-3) + \dfrac{2}{n} \times f(n-2) + 1$
依此类推，有：
$f(n - i) = \dfrac{n - i}{n} \times f(n - i - 1) + \dfrac{i}{n} \times f(n - i) + 1$，化简得到：
$\dfrac{n - i}{n} f(n-i) = \dfrac{n-i}{n} \times f(n-i-1) + 1$，继续化简得到：
$f(n - i) = f(n - i - 1) + \dfrac{n}{n-i}$，用 $x$ 替换 $n - i$ 得到：
$f(x) = f(x - 1) + \dfrac{n}{x}$

显然有 $f(1) = n$，递推即可。

这类题型相当常见，例如一样的题还有 [SHOI2002]百事世界杯之旅。

```cpp
#include <cstdio>
int T,n;
int main()
{
    scanf("%d",&T);
    while(T--)
    {
        scanf("%d", &n);
        double last = n, now = 0;
        for (int i = 2; i <= n; ++i)
        {
            now = last + n * 1.0 / i;
            last = now;
        }
        printf("%.2f\n", last);
    }
    return 0;
}
```

###  [USACO08OCT]Bovine Bones G

给三个骰子，范围分别为 $[1,a]$,$[1,b]$,$[1,c]$，求三个骰子上的数字和，哪个和出现最频繁。

虽然计算期望并不严谨，但依然可以试试。

数字在 $[1,a]$ 中平均分布，期望为 $\dfrac{a - 1}{2} + 1$，其余同理。

可以得到整体期望为 $\dfrac{a + b + c + 3}{2}$

然而问题在于，原题中的数字并非实数线性分布，而是限制在整数域内。而要计算的是出现最频繁的和，而非期望和。因此这个解法显然是有问题的。

明显的问题在于可能计算出非整数期望。

正确的方法应该是算事件数，即求 $x_1 + x_2 + x_3 = k$ 的方案数，如果不考虑限定范围的话，显然就是经典高中数学排列组合题，$k - 1$ 个间隔中插入两个隔板即可。

限定范围后，可能可以计算出不合法数目，然后容斥一下。

当然，原题数据范围奇小，直接枚举暴力即可。



### LuoguP1654 OSU!

给出每个位置为 $1$ 的概率，一整段长度为 $x$ 的 $1$ 对答案的贡献为 $x^3$，求期望得分。

定义 $p(i)$ 为位置 $i$ 为 $1$ 的概率。

考虑多一个 $1$ 对答案的贡献：$(x + 1)^3 - x^3 = 3x^2 + 3x + 1$，那么维护 $x^2$ 与 $x$ 的期望值即可进行转移。

定义 $f(i)$ 为结尾连续一段的 $x$ 的期望， $g(i)$ 为结尾连续一段的 $x^2$ 的期望，$h(i)$ 为得分的期望，有：
$f(i) = [f(i-1) + 1] \times p(i)$
$g(i) = [g(i-1) + 2f(i-1) + 1] \times p(i)$
$h(i) = h(i-1) + p(i) \times [3g(i-1) + 3f(i-1) + 1]$

```cpp
#include <cstdio>
const int maxn = 1e5 + 100;
int n;
double p[maxn], f[maxn], g[maxn], h[maxn];
int main()
{
    scanf("%d", &n);
    for (int i = 1; i <= n; ++i) scanf("%lf", p + i);
    for (int i = 1; i <= n; ++i)
    {
        f[i] = (f[i - 1] + 1) * p[i];
        g[i] = (g[i - 1] + 2 * f[i - 1] + 1) * p[i];
        h[i] = h[i - 1] + (3 * g[i - 1] + 3 * f[i - 1] + 1) * p[i];
    }
    printf("%.1f",h[n]);
    return 0;
}
```

### LuoguP4550 收集邮票

买第 $k$ 张邮票需要 $k$ 元，要收集全部邮票，求期望花费。
设 $f(i)$ 为现在取了 $i$ 张邮票，要取完剩下的邮票的期望次数。
初始有 $f(n) = 0$，考虑转移：
每次购买，有 $\dfrac{i}{n}$ 的概率取到有的，$\dfrac{n-i}{n}$ 的概率取到现在没有的，因此写出：
$f(i) = \dfrac{i}{n} \times f(i) + \dfrac{n-i}{n} \times f(i+1) + 1$，化简有：
$f(i) = f(i + 1) + \dfrac{n}{n-i}$，与上面的某题是一样的。

设 $g(i)$ 为现在取了 $i$ 张邮票，要取完剩下的邮票的期望花费。
初始有 $g(n) = 0$，考虑转移：
有 \dfrac{i}{n} 的概率取到有的，在此之后，期望还要取的次数是 $f(i)$，考虑顺着买邮票与倒着买是相同的，相当于在此之前，买了 $f(i)$ 张，此时花费为 $f(i) + 1$，因此：此时 $g(i) = \dfrac{i}{n} \times [g(i) + f(i) + 1]$
有 $\dfrac{n-i}{n}$ 的概率取到没有的，在此之后，期望还要取的次数为 $f(i+1)$，同理转化为买了 $f(i+1)$ 张，花费为 $f(i+1) + 1$，此时有：$g(i) = \dfrac{n-i}{n} \times [g(i+1) + f(i+1) + 1]$，合并起来有：
$g(i) = \dfrac{i}{n} \times [g(i) + f(i) + 1] + \dfrac{n-i}{n} \times [g(i+1) + f(i+1) + 1]$，化简得：

$g(i) = \dfrac{i}{n-i} \times f(i)+g(i+1)+f(i+1)+\dfrac{n}{n-i}$

转移即可。

```cpp
#include <cstdio>
const int maxn = 1e5 + 100;
int n;
double f[maxn],g[maxn];
int main()
{
    scanf("%d",&n);
    for(int i = n - 1;i>=0;--i)
    {
        f[i] = f[i + 1] + 1.0 * n / (1.0 * n - i);
        g[i] = (1.0 * i) / (n - i) * f[i] + g[i + 1] + f[i + 1] + (1.0 * n) / (n - i);
    }
    printf("%.2f",g[0]);
    return 0;
}
```

### LuoguP3802 小魔女帕琪

七种物品，每种有 $a_i$ 个，每次等概率取出一个。连续 $7$ 种不同的物品会触发一次终极魔法，而不同终极魔法对应的物品可以有重合。

求终极魔法触发的期望次数。

先考虑前七次就触发魔法的概率：

$\prod_{i=1}^7 \dfrac{a_i}{n - i + 1}$

而该 $a_i$ 的顺序如何排列是不会影响触发魔法的，因此 $7!$ 种排列都是可行的。

概率为：$7! \times \prod_{i=1}^7 \dfrac{a_i}{n - i + 1}$

考虑将前七次拓展到所有次数，可以发现，这七次连续抽取的段无论在哪里，概率都是上式。

这类似于抽签游戏，在条件概率下是公平的。严格证明如下：

假定第一次取走一个 $a_1$，剩余 $n - 1$ 个球，第 $2$ 至 $8$ 个抽取满足条件的概率为：

$\dfrac{a_1}{n} \times 7! \times \dfrac{a_2}{n - 1}\times \dfrac{a_3}{n - 2} \times \dfrac{a_4}{n - 3} \times \dfrac{a_5}{n - 4} \times \dfrac{a_6}{n - 5} \times \dfrac{a_7}{n - 6} \times \dfrac{a_1 - 1}{n - 7}$

其余同理，将第一次取走 $a_x$ 的概率累加得到：

$7! \times \prod _{i=1}^7 \dfrac{a_i}{n-i+1} \times \sum _{i=1}^7 \dfrac{a_i - 1}{n - 7}$

显然最后这部分的和为 $1$，因此取走第一个物品后，开头的前七次触发魔法的概率与未取时相等。

依此推广到取 $m$ 个物品时，概率都不变。直到只剩下 $7$ 个物品为止。

最后答案就是：

$7! \times \prod _{i=1}^7 \dfrac{a_i}{n-i+1} \times (n - 6)$

需要特判零的情况，避免除数为零。

```cpp
#include <cstdio>
using namespace std;
long long a[10];
int main()
{
    long long n = 0;
    for (int i = 1; i <= 7; ++i) scanf("%lld", a + i), n += a[i];
    double ans = 7 * 6 * 5 * 4 * 3 * 2 * 1 * (n - 6);
    if(n <= 6) 
    {
        puts("0.000");
        return 0;
    }
    for (int i = 1; i <= 7; ++i) ans = ans / (n - i + 1) * a[i];
    printf("%.3f", ans);
    return 0;
}
```

### LuoguP6154 游走

给一个有向无环图，随机选择一条路径，求路径期望长度。

假定有 $num$ 条路径，所有路径长度总和为 $len$，那么答案就是 $\dfrac{len}{num}$。

考虑如何求出 $num$ 与 $len$，定义 $f(i)$ 为以 $i$ 为结尾的路径条数，定义 $g(i)$ 为以 $i$ 结尾的路径长度和。

显然有 $f(v) += f(u)$，$g(v) += g(u) + f(u)$

拓扑排序转移即可。

```cpp
#include <cstdio>
#include <vector>
#include <algorithm>
#include <ctype.h>
const int bufSize = 1e6;
using namespace std;
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
#define int long long
const int mod = 998244353;
const int maxn = 1e5 + 100;
vector<int> E[maxn];
int n, m;
int f[maxn], g[maxn], in[maxn];
int q[maxn], qh, qt;
int fastpow(int x,int k)
{
    int res = 1;
    while (k)
    {
        if (k & 1) res = (1ll * res * x) % mod;
        x = (1ll * x * x) % mod;
        k >>= 1;
    }
    return res;
}
int inv(int x) { return fastpow(x, mod - 2); }
signed main()
{
    read(n), read(m);
    for (int i = 1; i <= m; ++i)
    {
        int x, y;
        read(x), read(y), E[x].push_back(y), in[y]++;
    }
    qh = 1, qt = 0;
    for (int i = 1; i <= n; f[i] = 1, ++i) if (in[i] == 0) q[++qt] = i;
    while(qt >= qh)
    {
        int u = q[qh++];
        for (vector<int>::iterator it = E[u].begin(); it != E[u].end(); ++it)
        {
            int v = *it;
            (f[v] += f[u]) %= mod;
            (g[v] += g[u] + f[u]) %= mod;
            if (--in[v] == 0) q[++qt] = v;
        }
    }
    int sf = 0,sg = 0;
    for (int i = 1; i <= n; ++i) sf = (sf + f[i]) % mod, sg = (sg + g[i]) % mod;
    printf("%lld\n", (1ll * sg * inv(sf)) % mod);
    return 0;
}
```