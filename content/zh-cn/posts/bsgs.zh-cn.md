---
title: "BSGS(大步小步算法)解离散对数问题"
date: 2020-06-13T14:58:04+08:00
slug: "bsgs"
type: posts
tags:
  - math
  - bsgs
categories:
  - Algorithm
---


全名为 BabyStepGiantStep 的算法，用于求解离散对数问题。

$$
A^x \equiv B \pmod{C}
$$

其中 $A,C$ 互质。

考虑暴力枚举， $x = 0$ 时 $A^x \equiv 1$ ，而由费马小定理， $A^{C-1} \equiv 1$ ，往后就出现重复。那么只需要枚举 $[0,C-1)$ 即可。

而利用分块思想可以简化计算。

设 $x = i \times \sqrt{C} - j,i \in [1,\sqrt{C}],j \in [0,\sqrt{C}]$

$A^{i \times \sqrt{C} - j} \equiv B \pmod{C}$

$\dfrac{A^{i \times \sqrt{C}}}{A^j} \equiv B \pmod {C}$

$\dfrac{A^{i \times \sqrt{C}}}{A^j \times B} \equiv 1 \pmod{C}$

即 $\bmod C$ 意义下， $A^{i \times \sqrt{C}} = A^j \times B$

预处理出所有 $A^j \times B$ 的值对应的 $j$ ，枚举 $i$ 即可快速查询到对应的 $j$ ，从而计算出解。

可以利用哈希表实现。

---

```cpp
#include <cstdio>
#include <list>
#include <cmath>
#include <cstring>
#define ll long long
ll C,A,B;
struct BetterListHashTable
{
    static const int mod = 1000103;
    static const int maxn = 1e6 + 100;
    struct node
    {
        int next, key, val;
    } A[maxn + 10];
    int head[mod + 100],tot;
    int hash(int x)
    {
        int res = x % mod;
        if (res < 0) res += mod;
        return res;
    }
    int & operator [](int key)
    {
        int pos = hash(key);
        for (int p = head[pos]; p; p = A[p].next)
            if (A[p].key == key) return A[p].val;
        A[++tot].next = head[pos], A[tot].key = key, A[tot].val = 0, head[pos] = tot;
        return A[tot].val;
    }
    int* find(int key)
    {
        int pos = hash(key);
        for(int p = head[pos];p;p=A[p].next)
            if(A[p].key == key) return &A[p].val;
        return NULL;
    }
}HT;
inline ll fastpow(ll x,ll k)
{
    ll res = 1;
    while(k)
    {
        if(k & 1) res = (res * x) % C;
        x = (x * x) % C;
        k >>= 1;
    }
    return res % C;
}
int main()
{
    scanf("%lld %lld %lld",&C,&A,&B);
    ll sqrtc = std::ceil(std::sqrt(C));
    ll t = B % C;
    for(int j = 0;j<=sqrtc;++j)
    {
        HT[t] = j;
        t = t * A % C;
    }
    ll sq = fastpow(A,sqrtc),ans;
    t = sq;
    for(int i = 1;i<=sqrtc;++i)
    {
        int *ans = HT.find(t);
        if(ans != NULL)
        {
            printf("%lld\n",i * sqrtc - *ans);
            return 0;
        }
        t = (t * sq) % C;
    }
    puts("no solution");
    return 0;
}
```

---

例题：[多少个 $1$ ？](https://b3logfile.com/siyuan/1609132319768/https://www.luogu.com.cn/problem/P4884)

给整数 $K$ 与质数 $m$ ，求最小的 $N$ 使得 $11\cdots111 (N 个 1) \equiv K \pmod{m}$

两边同时 $\times 9 + 1$ ，可以化为：

$10^N \equiv 9K + 1 \pmod{m}$

然后套上板子即可。

## exBSGS

$$
A^x \equiv B \pmod{P}
$$

然而当 $A,P$ 不互质时，就需要使用 EXBSGS.

设 $d_1 = \gcd(A,P)$ ，若 $d_1 \nmid B$ ，原方程无解。

证明：

$$
\begin{aligned} & A^x \equiv B \pmod {P} \\ & A \times A^{x-1} \equiv B \pmod{P} \\ & A \times A^{x-1} + P \times k = B\end{aligned}
$$

将 $A^{x-1},k$ 看做自变量，则当 $\gcd(A,P) \mid B$ 时该二元一次不定方程有解。

---

$$
\begin{aligned} 
& A^x \equiv B \pmod {P} 
\\
\Rrightarrow & \dfrac{A^x}{d_1} \equiv \dfrac{B}{d_1} \pmod{\dfrac{P}{d_1}}
\\
\Rrightarrow & \dfrac{a}{d_1} \times A^{x-1} \equiv \dfrac{B}{d_1} \pmod{\dfrac{P}{d_1}}
\end{aligned}
$$

若 $\gcd(A,\dfrac{P}{d_1}) \neq 1$ ，则继续除。

令 $D = \prod \limits_{i=1}^m d_i$

$$
\begin{aligned}& \dfrac{a^m}{D} \times A^{x-m} \equiv \dfrac{B}{D} \pmod{\dfrac{P}{D}} 
\\ \Rrightarrow & A^{x-m} \equiv \dfrac{B}{D} \times \dfrac{D}{a^m} \pmod{\dfrac{P}{D}}\\ \Rrightarrow & A^{x-m} \equiv \dfrac{B}{a^m}  \pmod{\dfrac{P}{D}}\end{aligned}
$$

此时 $\gcd(A,\dfrac{P}{D}) = 1$ ，可以使用 BSGS 求解。

随后加上 $m$ 即可。

---

```cpp
#include <cstdio>
#include <cmath>
#include <cstring>
#define ll long long
ll fastpow(ll x,ll k,ll mod)
{
    ll res = 1;
    for (; k; k >>= 1)
    {
        if (k & 1) res = (res * x) % mod;
        x = (x * x) % mod;
    }
    return ((res % mod) + mod) % mod;
}
struct BetterListHashTable
{
    static const int mod = 100103;
    static const int maxn = 1e6 + 100;
    struct node
    {
        int next, key, val;
    } A[maxn + 10];
    int head[mod + 100], tot;
    void clear() { memset(head, 0, sizeof(head)), tot = 0; }
    int hash(int x)
    {
        int res = x % mod;
        if (res < 0) res += mod;
        return res;
    }
    int& operator[](int key)
    {
        int pos = hash(key);
        for (int p = head[pos]; p; p = A[p].next)
            if (A[p].key == key) return A[p].val;
        A[++tot].next = head[pos], A[tot].key = key, A[tot].val = 0, head[pos] = tot;
        return A[tot].val;
    }
    int* find(int key)
    {
        int pos = hash(key);
        for (int p = head[pos]; p; p = A[p].next)
            if (A[p].key == key) return &A[p].val;
        return NULL;
    }
} HT;
inline ll BSGS(ll A, ll B, ll P)
{
    //A^x = B (mod P) (A,P) = 1
    HT.clear();
    ll t = B;
    ll sqrtp = std::ceil(std::sqrt(P));
    for (int i = 0; i <= sqrtp; ++i) HT[t] = i, t = (t * A) % P;
    ll sq = fastpow(A, sqrtp, P);
    t = sq;
    for (int i = 1; i <= sqrtp; ++i)
    {
        int* ans = HT.find(t);
        if (ans != NULL) return i * sqrtp - *ans;
        t = (t * sq) % P;
    }
    return -1;
}
ll gcd(ll a, ll b) { return b == 0 ? a : gcd(b, a % b); }
void exgcd(ll a, ll b, ll& x, ll& y)
{
    if (b == 0) return (void)(x = 1, y = 0);
    exgcd(b, a % b, y, x), y -= (a / b) * x;
}
ll inv(ll x, ll P)
{
    //xinv + Pk = 1
    ll k1, k2;
    exgcd(x, P, k1, k2);
    k1 = ((k1 % P) + P) % P;
    return k1;
}
inline ll exBSGS(ll A,ll B,ll P)
{
    ll m = 0,D = 1,g;
    if(B == 1) return 0;
    while((g = gcd(A,P)) != 1)
    {
        if (B % g) return -1;
        P /= g, ++m;
    }
    B = (B * inv(fastpow(A, m, P), P)) % P;  //inv(a^m)
    ll res = BSGS(A, B, P);
    if (res == -1) return -1;
    return res + m;
}
```

例题：[【模板】拓展 BSGS](https://b3logfile.com/siyuan/1609132319768/https://www.luogu.com.cn/problem/P4195)