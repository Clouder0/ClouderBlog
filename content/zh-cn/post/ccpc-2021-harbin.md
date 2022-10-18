---  
title: 'CCPC 2021 哈尔滨站'
date: 2022-10-18T22:30:58+08:00
slug: "ccpc-2021-harbin"  
type: post  
tags:  
  - ccpc
categories:  
  - Algorithm  
---  


补题。

## E

​#ACM/数学#​

$A_i = 2^{i-1} \bmod M$，给出 $A$，找出模数。如果模数不唯一，则无解。

对于第一个 $A_i \neq 2^{i-1}$，它被模了，而上一个数没有被模，说明 $M \in (2^{i-2},2^{i-1}]$，则可以得到 $A_i = 2^{i-1} - M$，从而有 $M = 2^{i-1} - A_i$，这里注意，$M > A_i$ 也是要成立的。

然后拿着这个模数检验整个序列，如果符合则是唯一解，否则无解。

```cpp
#include <cstdio>
#include <algorithm>
#include <ctype.h>
#define int long long
const int bufSize = 1e6;
inline char nc()
{
    #ifdef DEBUG
    return getchar();
    #endif
    static char buf[bufSize], *p1 = buf, *p2 = buf;
    return p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, bufSize, stdin), p1 == p2) ? EOF : *p1++;
}
inline void read(char *s)
{
    static char c;
    for (; !isalpha(c); c = nc());
    for (; isalpha(c); c = nc()) *s++ = c;
    *s = '\0';
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
const int maxn = 1e5 + 10;
int T, n, a[maxn];
signed main()
{
    read(T);
    while(T--)
    {
        read(n);
        for (int i = 1; i <= n; ++i) read(a[i]);
        long long M, now;
        for (int i = 1; i <= n; ++i)
        {
            if (a[i] > (1ll << (i - 1))) goto no;
            if (a[i] != (1ll << (i - 1)))
            {
                M = (1ll << (i - 1)) - a[i];
                if(a[i] >= M) goto no;
                now = a[i];
                for (int j = i + 1; j <= n; ++j)
                {
                    now = (now * 2) % M;
                    if (now != a[j]) goto no;
                }
                printf("%lld\n", M);
                goto yes;
            }
        }
        goto no;
    yes:
        continue;
    no:
        puts("-1");
        continue;
    }
    return 0;
}
```

## I

​#ACM/二进制#​ #ACM/思维#​

每次可以选出一个子序列，子序列中第 $i$ 个元素减去 $2^{i-1}$.  
一个元素可以在子序列中出现多次，也就是可以同时减 $1,2,4,\cdots$

要求最少次数后让整个序列为零。

肯定是与位有关的。

选出的子序列如果可以更长，则一定更长较优。

每个原序列中的元素，在一次操作中，相当于某一位减一。

扫描整个序列，记 $num(i)$ 为第 $i$ 位为 $1$ 的数量。

假设 $num(i) \ge num(i+1)$ 恒成立，则答案就是 $num(0)$  

而 $num(i)$ 中的 $1$ 可以转化为 $num(i-1)$ 中的 $2$.

反复扫描，直到 $num$ 单调不增，此时输出答案即可。

时间复杂度懒得证，但很真实。

```cpp
#include <cstdio>
#include <algorithm>
#include <ctype.h>
const int bufSize = 1e6;
#define int long long
inline char nc()
{
    #ifdef DEBUG
    return getchar();
    #endif
    static char buf[bufSize], *p1 = buf, *p2 = buf;
    return p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, bufSize, stdin), p1 == p2) ? EOF : *p1++;
}
inline void read(char *s)
{
    static char c;
    for (; !isalpha(c); c = nc());
    for (; isalpha(c); c = nc()) *s++ = c;
    *s = '\0';
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
const int maxn = 1e5 + 100;
int T,n,a[maxn], cnt[100];
signed main()
{
    read(T);
    while(T--)
    {
        read(n);
        for (int i = 0; i < 32; ++i) cnt[i] = 0;
        for (int i = 1; i <= n; ++i) read(a[i]);
        for (int i = 1; i <= n; ++i)
            for (int j = 0; (1 << j) <= a[i]; ++j) cnt[j] += ((a[i] >> j) & 1);
//        for(int i = 0;i<32;++i) printf("%d %d\n",i,cnt[i]);
        for (int t = 1; t < 100000; ++t)
        {
            int flag = 1;
            for (int i = 32; i; --i)
            {
                if (cnt[i] > cnt[i - 1])
                {
                    int give = (cnt[i] - cnt[i - 1] + 2) / 3;
                    cnt[i] -= give, cnt[i - 1] += give * 2;
                    flag = 0;
                }
            }
            if (flag) break;
        }
        printf("%lld\n", cnt[0]);
    }
    return 0;
}
```

## B

​#ACM/思维#​ #ACM/DP#​

对于序列 $A$ 的某个子序列，若满足 $A_{b_1} + A_{b_2} = A_{b_3} + A_{b_4} = \cdots$，则称其为一个魔法序列。

找出最长的魔法序列的长度。

值域很小，一个显然的做法是枚举 $sum$. 然后匹配该 $sum$ 下可以得到的最长魔法序列长度。

定义 $f(i)$ 为在前 $i$ 个元素中选出的最长魔法序列的长度。

$$
f(i) = \max f(j-1) + 2 \text{ where } sum - A(i)  = A(j)
$$

那么我们维护一个以 $A(j)$ 为下标，$\max f(j-1)$ 为值的数组，就可以做到单次 $O(n)$ 了。

```cpp
#include <cstdio>
#include <algorithm>
#include <ctype.h>
using namespace std;
const int bufSize = 1e6;
#define int long long
#define DEBUG
inline char nc()
{
    #ifdef DEBUG
    return getchar();
    #endif
    static char buf[bufSize], *p1 = buf, *p2 = buf;
    return p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, bufSize, stdin), p1 == p2) ? EOF : *p1++;
}
inline void read(char *s)
{
    static char c;
    for (; !isalpha(c); c = nc());
    for (; isalpha(c); c = nc()) *s++ = c;
    *s = '\0';
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
const int maxn = 2e5 + 100;
int n, a[maxn], maxx[maxn], f[maxn];
signed main()
{
    read(n);
    for (int i = 1; i <= n; ++i) read(a[i]);
    int res = 0;
    for (int sum = 2; sum <= 200; ++sum)
    {
        for (int i = 0; i <= 200; ++i) maxx[i] = -(1ll << 60);
        for (int i = 1; i <= n; ++i) f[i] = 0;
        maxx[a[1]] = 0;
        for (int i = 2; i <= n; ++i)
        {
            f[i] = f[i - 1];
            if (a[i] >= sum)
                continue;
            f[i] = max(maxx[sum - a[i]] + 2, f[i - 1]);
            maxx[a[i]] = max(maxx[a[i]], f[i - 1]);
            res = max(res, f[i]);
        }
    }
    printf("%lld\n", res);
    return 0;
}
```

## J

队友做的题，我也来补一下。

给一个 $n \times m$ 的矩阵，计算：

$$
\sum \limits _{i=1}^n \sum \limits _{j=1}^m [\min \{M_{u, v} \mid u=i \text{ or } v=j\} = M_{i,j}]
$$

也就是数满足 $M_{i,j}$ 同时为行列最小值的 $(i,j)$ 的个数。

有手就行。

```cpp
#include <algorithm>
#include <cstdio>
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
inline void read(char* s)
{
    static char c;
    for (; !isalpha(c); c = nc())
        ;
    for (; isalpha(c); c = nc()) *s++ = c;
    *s = '\0';
}
template <typename T>
inline T read(T& r)
{
    static char c;
    static int flag;
    flag = 1, r = 0;
    for (c = nc(); !isdigit(c); c = nc())
        if (c == '-') flag = -1;
    for (; isdigit(c); c = nc()) r = r * 10 + c - 48;
    return r *= flag;
}
const int maxn = 1100;
int n, m, a[maxn][maxn], mini[maxn], minj[maxn];
int main()
{
    read(n), read(m);
    for (int i = 1; i <= n; ++i)
        for (int j = 1; j <= m; ++j) read(a[i][j]);
    for (int i = 1; i <= n; ++i)
    {
        mini[i] = a[i][1];
        for (int j = 2; j <= m; ++j) mini[i] = std::min(mini[i], a[i][j]);
    }
    for (int j = 1; j <= m; ++j)
    {
        minj[j] = a[1][j];
        for (int i = 1; i <= n; ++i) minj[j] = std::min(minj[j], a[i][j]);
    }
    int res = 0;
    for (int i = 1; i <= n; ++i)
        for (int j = 1; j <= m; ++j) res += (a[i][j] == mini[i] && a[i][j] == minj[j]);
    printf("%d\n", res);
    return 0;
}
```

## D

​#ACM/枚举#​ #ACM/数学#​

给一个分数 $\dfrac{p}{q}$，上下删除某些数字后，值和原分数一致，这被称为一个合法结果。

输出 $p$ 最小的结果。

显然我们可以通过 $\gcd$ 化简得到真正的最简形 $\dfrac{a}{b}$，然后枚举 $p$ 的按位子集 $p'$（删掉几个数后的数字）。

要求 $\dfrac{p'}{q'} = \dfrac{a}{b}$，则 $q' = \dfrac{p'b}{a}$，若 $q'$ 是 $q$ 的子序列则合法，否则不合法。

可以先判断 $a \mid p'$ 剪枝。

时间复杂度大概是 $O(10 \times 18 \times  2^{18})$ 带上若干常数。考虑到有剪枝，凑合凑合，勉强勉强。

> 这破烂题目，题意属实不太清晰。
>
> 要求分子的数值最小。
>
> 删除某个数字的话，必须上下同时删除。删除的个数要相等。
>
> 注意删除后有前导零时，前导零可以省略。在检验能否删除到目标状态时，需要将删除的零的个数作为区间来进行判断。
>
> 使用 unsigned long long 完全足够，先除后乘避免溢出。
>

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;
#define int unsigned long long
int n, p, q, A, B, P[30], Q[30], lP, lQ;
int minLen, ansp, ansq;
int minn;
int num2arr(int num, int* arr)
{
    int p = 0;
    while (num) arr[++p] = num % 10, num /= 10;
    return p;
}
int arr2num(int* arr,int len)
{
    __int128 res = 0;
    for (int i = len; i; --i) res = res * 10 + arr[i];
    return res;
}
int gcd(int a, int b) { return b == 0 ? a : gcd(b, a % b); }
int subp[30], subq[30], erased[30], bin[10], bin2[10], leadZero1;
bool check(int a[], int b[],int la,int lb)
{
    for(int i = 0;i<=9;++i) bin2[i] = 0;
    int p = 1;
    for (int i = 1; i <= lb; ++i)
    {
        while(p <= la && a[p] != b[i]) 
        {
            bin2[a[p]]++;
            ++p;
        }
        if(p > la) return false;
        ++p;
    }
    int leadZero = 0;
    while(p <= la)
    {
        if (a[p] != 0)
            bin2[a[p]]++;
        else ++leadZero;
        ++p;
    }
    for(int i = 1;i<=9;++i) if(bin[i] != bin2[i]) return false;
    return bin[0] >= bin2[0]  && bin[0] <= bin2[0] + leadZero;
    return true;
}
void dfs(int pos, int len, int elen)
{
    if(pos == lP + 1)
    {
        if (!len) return;
        int nowp = arr2num(subp, len);
        if(nowp >= minn) return;
        if(nowp == 0) return;
//        printf("%llu\n",nowp);
        if (nowp % A != 0) return;
        int nowq = nowp / A * B;
        int lsq = num2arr(nowq, subq);
        for (int i = 0; i <= 9; ++i) bin[i] = 0;
        for(int i = 1;i<=elen;++i) bin[erased[i]]++;
        if (!check(Q, subq, lQ, lsq)) return;
        minn = nowp, minLen = len, ansp = nowp, ansq = nowq;
        return;
    }
    erased[elen + 1] = P[pos];
    dfs(pos + 1, len, elen + 1);
    subp[len + 1] = P[pos];
    dfs(pos + 1, len + 1, elen);
}
signed main()
{
    scanf("%llu", &n);
    while (n--)
    {
        minLen = 40;
        minn = 1ull << 63;
        scanf("%llu%llu", &p, &q);
        int g = gcd(p, q);
        if(g == 1)
        {
            printf("%llu %llu\n", p, q);
            continue;
        }
        A = p / g, B = q / g;
        lP = num2arr(p, P), lQ = num2arr(q, Q);
        dfs(1, 0, 0);
        printf("%llu %llu\n", ansp, ansq);
    }
    return 0;
}
```

## G

​#ACM/图论/最短路#​ #ACM/状态压缩#​ #ACM/DP#​ #ACM/期望#​

给一个正边权的无向图，从 $1$ 到 $n$.

有两种状态：骑行和步行。步行速度为 $t$，骑行速度为 $r$.

有 $k$ 辆单车，分别在 $a_i$ 点处，有 $\dfrac{p_i}{100}$ 的概率是坏的。当人发现一辆好的单车，就会骑上直达终点。

计算最小期望时间。

一共有 $k$ 辆车，有点像是旅行商问题。

记 $d(x,y)$ 为 $x \to y$ 的最小路程。显然 $d(x,y) = d(y,x)$.

状压 DP，以 $1$ 为访问过某单车，$f(S,i)$ 为访问过若干单车，最终到达单车 $i$ 的最小期望时间。这里暂且不管 $i$ 是好是坏。

显然若到了某单车且单车是好的，那么直接骑走是最优的。

由于我们需要的是从头开始的最优决策，所以应该从零开始搜索。

定义状态 $S$ 每位 $1$ 代表访问过的自行车。

定义 $f(S,i)$ 为状态 $S$ 下，从 $i$ 点到 $n$ 的最小期望时间。

若 $i$ 处车没坏，则直接骑车。否则，要么继续找车，要么走路。

$$
f(S,i) = (1-p(i)) \times \dfrac{d(a(i),n)}{r} +  p(i) \times \min \left( \dfrac{d(a(i),a(j))}{t} + f(S | 1 << j,j), \dfrac{a(i),n}{t} \right)
$$

```cpp
#include <algorithm>
#include <cstdio>
#include <ctype.h>
#include <queue>
using namespace std;
#define int long long
const int bufSize = 1e6;
inline char nc()
{
#ifdef DEBUG
    return getchar();
#endif
    static char buf[bufSize], *p1 = buf, *p2 = buf;
    return p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, bufSize, stdin), p1 == p2) ? EOF : *p1++;
}
inline void read(char* s)
{
    static char c;
    for (; !isalpha(c); c = nc())
        ;
    for (; isalpha(c); c = nc()) *s++ = c;
    *s = '\0';
}
template <typename T>
inline T read(T& r)
{
    static char c;
    static int flag;
    flag = 1, r = 0;
    for (c = nc(); !isdigit(c); c = nc())
        if (c == '-') flag = -1;
    for (; isdigit(c); c = nc()) r = r * 10 + c - 48;
    return r *= flag;
}
const int maxn = 2e5 + 100;
struct node
{
    int to, next, w;
} E[maxn];
int head[maxn];
inline void add(const int& x, const int& y, const int& w)
{
    static int tot = 0;
    E[++tot].next = head[x], E[tot].to = y, head[x] = tot, E[tot].w = w;
}
int T, R, n, m, k, a[maxn];
int d[19][maxn], vis[maxn];
void dij(int s, int dis[])
{
    for (int i = 1; i <= n; ++i) vis[i] = 0, dis[i] = 1ll << 60;
    dis[s] = 0;
    priority_queue<pair<int, int>> q;
    q.push(make_pair(-dis[s], s));
    while (!q.empty())
    {
        int u = q.top().second;
        q.pop();
        if (vis[u]) continue;
        vis[u] = 1;
        for (int p = head[u]; p; p = E[p].next)
        {
            int v = E[p].to;
            if (dis[v] > dis[u] + E[p].w)
            {
                dis[v] = dis[u] + E[p].w;
                q.push(make_pair(-dis[v], v));
            }
        }
    }
}
long double f[1 << 19][19], p[20], P[1 << 19];
long double dfs(int S, int u)
{
    if (f[S][u] > 0.00000001) return f[S][u];
    long double minn = 1.0 * d[u][n] / T;
    for (int i = 1; i <= k; ++i)
        if ((((1 << (i - 1))) & S) == 0)
        {
            minn = min(minn, 1.0 * d[u][a[i]] / T + dfs(S | (1 << (i - 1)), i));
        }
    f[S][u] = (1.0 - p[u]) * d[u][n] / R + p[u] * minn;
    return f[S][u];
}
signed main()
{
    read(T), read(R);
    read(n), read(m);
    for (int i = 1, u, v, w; i <= m; ++i) read(u), read(v), read(w), add(u, v, w), add(v, u, w);
    read(k);
    for (int i = 1, t; i <= k; ++i) read(a[i]), read(t), p[i] = t / 100.0;
    dij(1, d[0]);
    for (int i = 1; i <= k; ++i) dij(a[i], d[i]);
    if (d[0][n] > 1e10) return puts("-1"), 0;
    long double res = 1.0 * d[0][n] / T;
    for (int i = 1; i <= k; ++i) res = min(res, dfs(1 << (i - 1), i) + 1.0 * d[0][a[i]] / T);
    printf("%.6Lf\n", res);
    return 0;
}
```

## C

​#ACM/DP#​ #ACM/DP/树形DP#​ #ACM/启发式合并#​

场上我在尝试写这题暴力时失败了。

给一棵树，初始时没有颜色，每次操作将 $u$ 为根的子树染成相同颜色，覆盖性染色。

问将**每个叶子**染成目标颜色所需的最少次数。

$n \le 10^5$.

对于每个叶子 $i$，其祖先中第一个染色节点必须与其目标颜色一致。

定义 $f(i,j)$ 为以 $i$ 为根的子树、在 $i$ 处染颜色 $j$、使子树内所有的叶子都符合预期所需的最少操作次数（不包括当前次染色）。

特别的，$f(i,0)$ 为在 $i$ 处不染色的最少所需操作次数。

定义 $minn(i)$ 为在 $i$ 处任意染色所需的最少操作次数。

$$
f(i,j) = 1 + \sum \min(f(v,j) - 1, minn(i))
$$

而：

$$
f(i,0) = \sum minn(i)
$$

可以再次简化，将 $f(u,0)$ 与 $minn(u)$ 做一个合并。

这就得到了一个朴素的 $O(n^2)$ 算法。

```cpp
int f[maxn][maxn], minn[maxn];
void dfs(int u)
{
    if(c[u]) return;
    for(int j = 1;j<=m;++j) f[u][j] = 1;
    for (int p = head[u]; p; p = E[p].next)
    {
        int v = E[p].to;
        dfs(v);
        for (int j = 1; j <= m; ++j) f[u][j] += std::min(minn[v], f[v][j] - 1);
        minn[u] += minn[v];
    }
    for (int j = 1; j <= m; ++j) minn[u] = std::min(minn[u], f[u][j]);
}
```

接下来考虑优化。显然只有某些颜色的值是有意义的。

考虑启发式合并，可以发现若某个颜色不存在，则加 $minn(v)$，否则加 $\min (minn(v), f(v,c))$. 那么一开始加 $minn(v)$，之后合并时加 $\min(minn(v),f(v,c)) - minn(v)$ 即可。

用一个加法懒标记来实现。需要特别处理重儿子的情况。

直接将重儿子的 map 挪过来，那么需要继承对应的 addtag. 重儿子的值也受其他子树影响，需要加上相应的 $minn(v)$，但不受自己影响，减去自己的 $minn(son)$.

其他子树却受重儿子影响，则在初始化颜色时补齐差值即可。

同时，直接继承重儿子的 $f$ 数组之后，还要做一个与 $minn(son)$ 取 $\min$ 的操作。可以利用 set 来实现。

复杂度大概是 $O(n\log^2 n)$ 吧。常数巨大。

```cpp
#include <algorithm>
#include <cstdio>
#include <cstring>
#include <ctype.h>
#include <map>
#include <set>
#define DEBUG
#define int long long
using namespace std;
const int bufSize = 1e6;
inline char nc()
{
#ifdef DEBUG
    return getchar();
#endif
    static char buf[bufSize], *p1 = buf, *p2 = buf;
    return p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, bufSize, stdin), p1 == p2) ? EOF : *p1++;
}
inline void read(char* s)
{
    static char c;
    for (; !isalpha(c); c = nc())
        ;
    for (; isalpha(c); c = nc()) *s++ = c;
    *s = '\0';
}
template <typename T>
inline T read(T& r)
{
    static char c;
    static int flag;
    flag = 1, r = 0;
    for (c = nc(); !isdigit(c); c = nc())
        if (c == '-') flag = -1;
    for (; isdigit(c); c = nc()) r = r * 10 + c - 48;
    return r *= flag;
}
const int maxn = 2e5 + 100;
int n, m, fa[maxn], c[maxn], leafnum;
struct node
{
    int to, next;
} E[maxn];
int head[maxn];
inline void add(const int& x, const int& y)
{
    static int tot = 0;
    E[++tot].next = head[x], E[tot].to = y, head[x] = tot;
}
int minn[maxn];
/*
int f[maxn][maxn], minn[maxn];
void dfs(int u)
{
    if(c[u]) return;
    for (int p = head[u]; p; p = E[p].next)
    {
        int v = E[p].to;
        dfs(v);
        for (int j = 1; j <= m; ++j) f[u][j] += std::min(minn[v],f[v][j]);
        minn[u] += minn[v];
    }
    for (int j = 1; j <= m; ++j) minn[u] = std::min(minn[u], f[u][j] + 1);
}
*/
int addtag[maxn];
map<int, int> mp[maxn];
set<pair<int, int>> S[maxn];
void dfs2(int u)
{
    if (c[u]) return;
    int son = 0;
    for (int p = head[u]; p; p = E[p].next)
    {
        int v = E[p].to;
        dfs2(v);
        if (!son || mp[v].size() > mp[son].size()) son = v;
        minn[u] += minn[v];
        addtag[u] += minn[v];
    }
    std::swap(mp[u], mp[son]);
    std::swap(S[u], S[son]);
    for (auto it = S[u].lower_bound(make_pair(minn[son] - addtag[son], 0)); it != S[u].end(); ++it)
        mp[u].erase(it->second);
    S[u].erase(S[u].lower_bound(make_pair(minn[son] - addtag[son], 0)), S[u].end());
    //    mp[u] = mp[son];
    addtag[u] += addtag[son];
    addtag[u] -= minn[son];
    for (int p = head[u]; p; p = E[p].next)
    {
        int v = E[p].to;
        if (v == son) continue;
        for (map<int, int>::iterator it = mp[v].begin(); it != mp[v].end(); ++it)
        {
            map<int, int>::iterator cur = mp[u].find(it->first);
            if (cur == mp[u].end())
            {
                mp[u][it->first] = std::min(minn[v], it->second + addtag[v]) - minn[v] + minn[son] - addtag[son];
                S[u].insert(make_pair(mp[u][it->first], it->first));
                minn[u] = min(minn[u], addtag[u] + mp[u][it->first] + 1);
            }
            else
            {
                S[u].erase(make_pair(cur->second, cur->first));
                cur->second += min(minn[v], it->second + addtag[v]) - minn[v];
                S[u].insert(make_pair(cur->second, cur->first));
                minn[u] = min(minn[u], addtag[u] + cur->second + 1);
            }
        }
    }
    //    printf("minn[%lld]=%lld\n",u,minn[u]);
}
signed main()
{
    read(n);
    for (int i = 2; i <= n; ++i) read(fa[i]), add(fa[i], i);
    for (int i = 1; i <= n; ++i)
    {
        read(c[i]);
        m = std::max(m, c[i]);
    }
    for (int i = 1; i <= n; ++i)
    {
        if (!c[i]) continue;
        //        for (int j = 0; j <= m; ++j) f[i][j] = n + 1;
        // f[i][c[i]] = 0;
        minn[i] = 1;
        mp[i][c[i]] = 0;
        S[i].insert(make_pair(0, c[i]));
    }
    // dfs(1);
    dfs2(1);
    //    for(int i = 1;i<=n;++i) for(int j = 1;j<=m;++j) printf("f[%lld][%lld]=%lld\n",i,j,f[i][j]);
    //     for (int i = 1; i <= n; ++i)
    //         for (map<int, int>::iterator it = mp[i].begin(); it != mp[i].end(); ++it)
    //             printf("f[%lld][%lld]=%lld\n", i, it->first, it->second + addtag[i]);
    printf("%lld\n", minn[1]);
    return 0;
}

```

## H

给两个数组 $S,T$ 与一个定值 $k$.

每次操作，可以选取一个 $a$，交换 $S(a,a+k-1)$ 与 $S(a+k,a+2k-1)$ 这两段区间。

问是否能通过有限次操作，将 $S$ 变成 $T$.

首先将 $S$ 的下标模 $k$ 分类，记为 $A_0,A_1,\cdots,A_{k-1}$.

每次操作，实质上是交换了 $A_i$ 中相邻两个数字的顺序。

更一般的，是交换 $A[0,i]$ 中第 $j,j+1$ 个元素与 $A[i+1,k-1]$ 中第 $j-1,j$ 个元素。

将 $T$ 模 $k$ 分类，记为 $B_0,\cdots,B_{k-1}$  

若 $A_i$ 中元素忽略顺序后与 $B_i$ 中元素不同，直接不可能。

每次操作进行交换，会同时改变所有组的逆序数奇偶性。

那我们求出由 $A_i,B_i$ 逆序数的奇偶性组成的序列，要么相同，要么相反，否则无解。

注意，如果某个序列有重复值，其逆序数就可以视作可奇可偶。

显然操作是可逆的，让 $A_i$ 经过变换成为单调不增的序列即可。

进行 $a=uk+i,a=uk+i+1$ 两次操作，即可将 $uk+i,(u+1)k+i,(u+2)k+i$ 进行一次轮转。通过类似于冒泡的手法，即可使数组有序。

以上两个结论即为充要条件。

不看题解根本想不到系列。

```cpp
#include <algorithm>
#include <cstdio>
#include <ctype.h>
#include <vector>
using namespace std;
const int bufSize = 1e6;
inline char nc()
{
#ifdef DEBUG
    return getchar();
#endif
    static char buf[bufSize], *p1 = buf, *p2 = buf;
    return p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, bufSize, stdin), p1 == p2) ? EOF : *p1++;
}
inline void read(char* s)
{
    static char c;
    for (; !isalpha(c); c = nc())
        ;
    for (; isalpha(c); c = nc()) *s++ = c;
    *s = '\0';
}
template <typename T>
inline T read(T& r)
{
    static char c;
    static int flag;
    flag = 1, r = 0;
    for (c = nc(); !isdigit(c); c = nc())
        if (c == '-') flag = -1;
    for (; isdigit(c); c = nc()) r = r * 10 + c - 48;
    return r *= flag;
}
#define int long long
const int maxn = 5e5 + 100;
int n, llen, rlen, S[maxn], T[maxn], k;
vector<int> A[maxn], B[maxn];
int num1[maxn], num2[maxn];
int H1[maxn], H2[maxn], cnt1, cnt2;
int tree[maxn], timetag[maxn], nowtime;
int duplicate[maxn];
int flag1, flag2;
long long res;
inline int lowbit(const int& x) { return x & (-x); }
inline void clear() { ++nowtime; }
inline void check(int x)
{
    if (timetag[x] < nowtime) tree[x] = 0, timetag[x] = nowtime;
}
void add(int x, int k, int len)
{
    for (; x <= len; x += lowbit(x))
    {
        check(x);
        tree[x] += k;
    }
}
int ask(int x)
{
    int res = 0;
    for (; x > 0; x -= lowbit(x)) check(x), res += tree[x];
    return res;
}

signed main()
{
    read(n);
    while (n--)
    {
        read(llen);
        for (int i = 1; i <= llen; ++i) read(S[i]);
        read(rlen);
        for (int i = 1; i <= rlen; ++i) read(T[i]);
        read(k);
        if (llen != rlen) goto no;
        for (int i = 0; i < k; ++i) A[i].clear(), B[i].clear(), duplicate[i] = 0;
        for (int i = 1; i <= llen; ++i) A[i % k].push_back(S[i]), B[i % k].push_back(T[i]);
        for (int i = 0; i < k; ++i)
        {
            cnt1 = cnt2 = 0;
            for (auto it = A[i].begin(); it != A[i].end(); ++it)
                H1[++cnt1] = *it;
            sort(H1 + 1, H1 + cnt1 + 1);
            for (auto it = B[i].begin(); it != B[i].end(); ++it)
                H2[++cnt2] = *it;
            sort(H2 + 1, H2 + cnt2 + 1);
            for (int i = 1; i <= cnt1; ++i)
                if (H1[i] != H2[i]) goto no;
            cnt1 = unique(H1 + 1, H1 + cnt1 + 1) - H1 - 1;
            if (cnt1 != cnt2) duplicate[i] = 1;
            for (auto it = A[i].begin(); it != A[i].end(); ++it)
                *it = lower_bound(H1 + 1, H1 + 1 + cnt1, *it) - H1;
            for (auto it = B[i].begin(); it != B[i].end(); ++it)
                *it = lower_bound(H1 + 1, H1 + 1 + cnt1, *it) - H1;
            res = 0;
            for (auto it = A[i].begin(); it != A[i].end(); ++it)
            {
                res += ask(*it - 1);
                add(*it, 1, cnt1);
            }
            num1[i] = res & 1;
            clear();
            res = 0;
            for (auto it = B[i].begin(); it != B[i].end(); ++it)
            {
                res += ask(*it - 1);
                add(*it, 1, cnt1);
            }
            num2[i] = res & 1;
            clear();
        }
        flag1 = 1, flag2 = 1;
        for (int i = 0; flag1 && i < k; ++i)
            if (!duplicate[i] && num1[i] != num2[i]) flag1 = 0;
        for (int i = 0; flag2 && i < k; ++i)
            if (!duplicate[i] && num1[i] == num2[i]) flag2 = 0;
        if (flag1 || flag2) goto yes;
        goto no;
    yes:
        puts("TAK");
        continue;
    no:
        puts("NIE");
        continue;
    }
    return 0;
}
```
