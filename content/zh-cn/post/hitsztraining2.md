---  
title: '校内选拔赛补题'
date: 2022-10-24T01:58:33+08:00
slug: "hitsztraining2"  
type: post  
tags:  
categories:  
  - Algorithm  
---  


被队友带的一场比赛。

只能说我的水平还是太低了，这种题都没法场上过掉。或者说我和 CF 终究是八字不合。

## CF1312D

给 $n,m$，求序列方案数，满足 $a_i \in [1,m]$，且 $a$ 严格单峰，$a$ 存在且仅存在一对相同的元素。

推式子的题目。

考虑峰值为 $p$，还需要 $n-1$ 个数，$n-2$ 个不同的值，从 $[1,p-1]$ 中选出相同值后，选其他值：$\dbinom{p-2}{n-3}$，然后排列在两侧。每个数要么在左侧，要么在右侧，而相同的值必须在异侧。

也就是说，我们可以决定剩下的 $n-3$ 个数的左右位置。显然分配完之后，左侧、右侧的顺序是确定的。

$$
\sum \limits _{p=1}^m (p-1) \times \dbinom{p-2}{n-3} \times 2^{n-3}
$$

特判一下，$n=2$ 时无解。

这里需要快速计算组合数，考虑 $\dbinom{a}{b} = \dfrac{a!}{(a-b)!b!}$，计算阶乘、阶乘逆元即可。

---

其实这个式子可以进一步简化。

考虑选出 $n-2$ 个值后，最大值一定是峰值，钦定某个非最大值为重复值，然后其余值分为在左在右。

$$
(n-2) \times \dbinom{m}{n-1} \times 2^{n-3}
$$

## CF1187D

 场上口胡了一个鬼畜的三维偏序。

首先可以考虑处理出每个 $a_i$ 应当到达的位置 $to(i)$.

每次选取相邻两个元素，实际上就可以实现类似于冒泡排序的效果。可以证明选择更长区间实现的操作，反复小操作同样能实现。

那么就只考虑相邻交换。

假设 $to(i) < i$，则要求 $i$ 一路能向左交换到 $to(i)$，路上遇到的值都要 $>a_i$ 才可以。特别的，如果某个 $a_j < a_i$，但是 $to(j) < to(i)$，那么可以先将 $a_j$ 往前挪走，不会对 $a_i$ 到达目标产生影响。

所以计数：

$$
\begin{cases}
to(i) < j < i \\
to(j) > to(i) \\
a_j < a_i
\end{cases}
$$

只要该数值非 $0$，则一定存在无法移动到目标位置的 $a_i$.

可以发现这类似于一个三维偏序问题。

另一种情况，若 $to(i) > i$，则 $i$ 向右移动，计数：

$$
\begin{cases}
i < j < to(i) \\
to(j) < to(i) \\
a_j > a_i
\end{cases}
$$

同样要求为零。

考虑到有不等式 $to(i) \ge i$ 恒成立，则若 $to(j) < to(i)$，$j < to(i)$ 也成立。因此可以转化为：

$$
\begin{cases}
j < i \\
to(i) < to(j) \\
a_j < a_i
\end{cases}
$$

$$
\begin{cases}
i < j  \\
to(j) < to(i) \\
a_i < a_j
\end{cases}
$$

都不成立。  
事实上，可以发现，这二者的结构多么类似啊！那其实我们只统计一侧就可以了。

$$
\begin{cases}
j < i \\
to(i) < to(j) \\
a_i \ge a_j
\end{cases}
$$

恒成立。

从左往右遍历 $i$，查询 $to(j) \in [to(i), n]$ 的对应 $a_j$ 最大值 $maxx$，如果 $maxx > a_i$ 成立则无解。  
然后将 $i$ 插入对应位置。

实际上因为这要求的是存在性问题，而不是严格的偏序计数，因此可以用这种方法解决。

（马上复习 cdq 分治）

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
const int maxn = 3e5 + 100;
int T, n, a[maxn], b[maxn], to[maxn];
vector<int> Va[maxn], Vb[maxn];
int f[maxn], tag[maxn], now;
inline void check(int x)
{
    if (tag[x] != now) f[x] = 1 << 30, tag[x] = now;
}
void update(int x, int k)
{
    //    printf("update %d %d\n",x,k);
    for (; x > 0; x -= (x & -x)) check(x), f[x] = std::min(f[x], k);
}
int ask(int x)
{
    int res = 1 << 30;
    for (; x <= n; x += (x & -x)) check(x), res = std::min(f[x], res);
    return res;
}
int main()
{
    read(T);
    while (T--)
    {
        read(n);
        for (int i = 1; i <= n; ++i) Va[i].clear(), Vb[i].clear();
        for (int i = 1; i <= n; ++i) read(a[i]), Va[a[i]].push_back(i);
        for (int i = 1; i <= n; ++i) read(b[i]), Vb[b[i]].push_back(i);
        for (int i = 1; i <= n; ++i)
        {
            if (Va[i].size() != Vb[i].size()) goto no;
            for (auto it = Va[i].begin(), it2 = Vb[i].begin(); it != Va[i].end(); ++it, ++it2)
                to[*it] = *it2;
        }
        //        for(int i = 1;i<=n;++i) printf("%d ",to[i]);
        //        puts("");
        ++now;
        for (int i = 1; i <= n; ++i)
        {
            //            printf("to %d\n",i);
            //            printf("ask %d %d\n", to[i] + 1,ask(to[i] + 1));
            if (ask(to[i] + 1) < a[i]) goto no;
            update(to[i], a[i]);
        }
        puts("YES");
        continue;
    no:
        puts("NO");
    }
    return 0;
}
```

## CF1718A2

给一个数组 a，选择区间 $[l,r]$，花费 $\lceil \dfrac{r-l+1}{2}\rceil$ 的代价，使区间内数 $\oplus x$，求清零的最小代价。

要么进行长度为一的区间操作，要么进行长度为二的区间操作。任何更长的操作都可以由这两种操作等价，而性价比不如。

如果 $[l,l+1]$ 操作了，那再对 $l+1$ 单独操作，和对 $l,l+1$ 分别单独操作是等价的。

那么可以认为长度为一的操作与长度为二的操作不相交。

但长度为二的操作是可以相交的。连续进行长度为二的操作，从 $l$ 到 $r$ 后，$a_r = a_l\oplus  a_{l+1} \oplus \cdots \oplus a_r$，一共进行 $r-l$ 次。如果此时 $a_r = 0$，就可以节约一次。

那么我们的可选的操作就是：

* 进行长度为一的操作，花费 $1$ 代价消掉 $1$ 个。
* 进行 $k$ 次长度为二的操作。花费 $k$ 的代价，消掉 $k$ 或 $k+1$ 个。当 $a_l\oplus  a_{l+1} \oplus \cdots \oplus a_r = 0$ 时即可消掉 $k+1$ 个。

那我们不断寻找异或和为零的段贪心即可。

```cpp
#include <cstdio>
#include <algorithm>
#include <set>
#include <ctype.h>
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
const int maxn = 1e5  + 100;
int T,n, a[maxn];
set<int> S;
int main()
{
    read(T);
    while(T--)
    {
        read(n);
        int zeronum = 0, pre = 0, res = n;
        for (int i = 1; i <= n; ++i) read(a[i]), zeronum += !a[i];
        //        for(int i = 1;i<=n;++i) printf("%d ",pre[i]);
        //        puts("");
        if (zeronum == n)
        {
            puts("0");
            goto end;
        }
        S.clear();
        S.insert(0);
        for (int i = 1; i <= n; ++i) 
        {
            pre ^= a[i];
            if (S.find(pre) != S.end())
                --res, S.clear(), S.insert(0), pre = 0;
            else S.insert(pre);
        }
        printf("%d\n", res);
        end:;
    }
    return 0;
}
```

## CF715B

给一张非负权图，可以修改 $0$ 权边，修改后所有边权为正数，问能否修改后使 $s \to t$ 的最短路长度为 $L$.

如果存在任意不经过 $0$ 权边的 $s \to t$ 路径，满足长度小于 $L$，则不可能。

然后加入零权边，把零权边改权为 $1$，跑出最短路如果 $\le L$ 则有解。

接下来我们要分配边。

不在最短路上的边一律分配无穷大。接下来就是要在最短路上做一个分配。

可以进行反复调整，每次将当前最短路通过一条零权边调整到 $L$，其余边依然为 $1$，然后再跑最短路，再次调整。

复杂度是 $O(nm\log n)$ 的。

还有一种复杂度更优的方法。

我们将第一到第 $p$ 条零权边设置为 $1$，剩下的无穷大，跑最短路。如果 $>L$，则 $p:=p-1$，一定可以找到唯一一条关键边，使得这条边若无穷大则无解，若 $1$ 则有解。

那么通过调整这条关键边，我们就能改变最短路的长度，从而找到一个解。

二分这条关键边的长度即可。

题解的二分真是灵活至极啊。

```cpp
#include <algorithm>
#include <cstdio>
#include <ctype.h>
#include <queue>
using namespace std;
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
const int maxn = 1e5 + 100;
int n, m, L, s, t;
struct node
{
    int to, next, val;
} E[maxn * 3];
int head[maxn], oldhead[maxn], tot;
inline void add(const int& x, const int& y, const int& w)
{
    E[++tot].next = head[x], E[tot].to = y, E[tot].val = w, head[x] = tot;
}
int dis[maxn], vis[maxn];
int U[maxn], V[maxn], W[maxn];
void dij()
{
    priority_queue<pair<int, int>> q;
    for (int i = 1; i <= n; ++i) dis[i] = 1ll << 40, vis[i] = 0;
    dis[s] = 0;
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
            if (dis[v] > dis[u] + E[p].val)
            {
                dis[v] = dis[u] + E[p].val;
                q.push(make_pair(-dis[v], v));
            }
        }
    }
}
signed main()
{
    read(n), read(m), read(L), read(s), read(t);
    ++s, ++t;
    for (int i = 1; i <= m; ++i)
    {
        read(U[i]), read(V[i]), read(W[i]);
        ++U[i], ++V[i];
        if (W[i] != 0) add(U[i], V[i], W[i]), add(V[i], U[i], W[i]);
    }
    dij();
    if (dis[t] < L)
    {
        puts("NO");
        return 0;
    }
    int l = 1, r = m, mid, ans = 0;
    while (l <= r)
    {
        mid = (l + r) >> 1;
        int old = tot;
        for (int i = 1; i <= n; ++i) oldhead[i] = head[i];
        for (int i = 1; i <= mid; ++i)
            if (!W[i]) add(U[i], V[i], 1), add(V[i], U[i], 1);
        dij();
        if (dis[t] > L)
            l = mid + 1;
        else
            ans = mid, r = mid - 1;
        tot = old;
        for (int i = 1; i <= n; ++i) head[i] = oldhead[i];
    }
    if(ans == 0)
    {
        puts("NO");
        return 0;
    }
    for (int i = 1; i < ans; ++i)
        if (!W[i]) add(U[i], V[i], 1), add(V[i], U[i], 1);
    //    printf("ans%lld\n",ans);
    l = 1, r = L;
    int res = 1;
    while (l <= r)
    {
        int old = tot;
        for (int i = 1; i <= n; ++i) oldhead[i] = head[i];
        mid = (l + r) >> 1;
        add(U[ans], V[ans], mid);
        add(V[ans], U[ans], mid);
        dij();
        if (dis[t] >= L)
            res = mid, r = mid - 1;
        else
            l = mid + 1;
        tot = old;
        for (int i = 1; i <= n; ++i) head[i] = oldhead[i];
    }
    puts("YES");
    for (int i = 1; i <= m; ++i)
    {
        if (W[i])
            printf("%lld %lld %lld\n", U[i] - 1, V[i] - 1, W[i]);
        else
        {
            if (i < ans)
                printf("%lld %lld 1\n", U[i] - 1, V[i] - 1);
            else if (i == ans)
                printf("%lld %lld %lld\n", U[i] - 1, V[i] - 1, res);
            else
                printf("%lld %lld %lld\n", U[i] - 1, V[i] - 1, L + 1);
        }
    }
    return 0;
}
```

## 1695D2

考虑链的情况，一个点就能确定一条链。

现在在链上再挂上一条链，形成一个根节点度为三的树。两个点就能确定任意点。

现在回到树上考虑。

对于某棵子树，对于其父亲来说，它内部有多少个询问点是没有意义的，因此可以将其抽象为一个点。

考虑一个类似换根的想法，以当前 $u$ 为根，如果其父亲子树中有询问点、$u$ 有 $k$ 个儿子，如果恰好有一个儿子子树是一整条链，那么只需要其余儿子子树内有询问点、父亲子树中有询问点，就可以确定这条链上任意点。

将询问点放在叶子上总是比较好的。

那么初始化时，我们需要叶子个数个询问点，就一定能确定任意点。一个询问点确定往上的一条链，而当两个询问点确定的链在祖先处汇聚，可以将其等价于一个询问点，从而递归、归纳证明可确定任意点。

而如果有某个点，其有一个儿子子树恰好为一条链，如果有其他的儿子，那么就恰好可以节约这条链端的询问点。

因为只要其余儿子子树内都有询问点，就可以将其等价为一条链挂上一个询问点，从而通过两点确定任意点。

但如果没有直链儿子，就不能节约。因为如果某棵子树内没有询问点，则一定可以找到一个分叉点 $v$，使 $v$ 父亲子树内询问点对 $v$ 内部深度相同的点给出相同距离。

那么从所有叶子出发，找到往上的第一个分叉点，节约之。

特判链与单点。

```cpp
#include <cstdio>
#include <algorithm>
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
const int maxn = 4e5 + 100;
int T,n;
struct node
{
    int to, next;
} E[maxn << 1];
int head[maxn],tot;
inline void add(const int &x,const int &y)
{
    E[++tot].next = head[x],E[tot].to = y,head[x] = tot;
}
int deg[maxn];
int q[maxn], qh, qt;
int main()
{
    read(T);
    while(T--)
    {
        read(n);
        tot = 0;
        for (int i = 1; i <= n; ++i) head[i] = 0, deg[i] = 0;
        for (int i = 1, u, v; i < n; ++i) read(u), read(v), add(u, v), add(v, u), ++deg[u], ++deg[v];
        int maxx = 0, res = 0;
        if(n == 1)
        {
            puts("0");
            goto end;
        }
        for (int i = 1; i <= n; ++i) maxx = std::max(maxx,deg[i]);
        if(maxx <= 2) 
        {
            puts("1");
            goto end;
        }
        qh = 1,qt = 0;
        for (int i = 1; i <= n; ++i)
            if (deg[i] == 1)
                ++res, q[++qt] = i;
        while(qt >= qh)
        {
            int u = q[qh++];
            deg[u] = 0;
            for(int p = head[u];p;p=E[p].next)
            {
                int v = E[p].to;
                if (deg[v] >= 3)
                    deg[v] = -1, --res;
                else if(deg[v] == 2)
                    q[++qt] = v;
            }
        }
        printf("%d\n", res);
        end:;
    }
    return 0;
}
```
