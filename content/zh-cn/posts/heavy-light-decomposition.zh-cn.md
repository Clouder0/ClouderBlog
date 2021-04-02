---
title: "轻重链剖分练习笔记"
date: 2020-10-21T12:08:52+08:00
slug: "heavydecomposition"
type: posts
tags:
  - heavydecomposition
  - tree
categories:
  - algorithm
---


## 前言

轻重链剖分，常被称为树链剖分，是一种常用的维护树上信息的算法。
它以子树大小为依据，将节点划分为重儿子与轻儿子，从而使整棵树被剖分成若干条重链。
每个轻儿子都是一条重链的开始。一个节点只在一条重链上。
从树上任意一点到根节点，最多经过 $\log n$ 条连续的链。
利用这样的特殊性质，可以解决许多问题。
由于是练习笔记，本文不再赘述概念有关内容。

## 练习题

### CF383C Propagating tree

[Link](https://codeforces.com/problemset/problem/383/C)

给一棵树，点有初始权值。现在有修改与查询操作。
修改操作使 $x$ 的权值增加 $val$，使 $x$ 的儿子的权值减少 $val$，使 $x$ 的儿子的儿子的权值增加 $val \ldots$

由于只操作子树，并不需要剖分性质，可以直接维护 dfs 序与子树大小。
修改操作相当于一次区间操作，思考如何实现增减的变化。
可以发现，正负与深度的奇偶有关。
维护一个数据结构，定义下标为奇数的位置的值为该点的额外权值，为偶数的位置的值为该点的额外权值的相反数，那么就可以用一次区间加来实现了。
初始权值可以直接单独记录下来，也可以初始化的时候插入数据结构中。

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
struct node
{
    int to,next;
}E[maxn << 1];
int head[maxn];
inline void add(const int &x,const int &y)
{
    static int tot = 0;
    E[++tot].next = head[x],E[tot].to = y,head[x] = tot;
}
int n, m, a[maxn], dep[maxn], size[maxn], in[maxn], out[maxn], ind, sum[maxn];
inline int lowbit(int x) { return x & -x; }
void ins(int x,int k) { for (; x <= n; x += lowbit(x)) sum[x] += k; }
int ask(int x)
{
    int res = 0;
    for (; x > 0; x -= lowbit(x)) res += sum[x];
    return res;
}
void dfs(int u,int fa)
{
    in[u] = ++ind, size[u] = 1;
    for (int p = head[u]; p; p = E[p].next)
    {
        int v = E[p].to;
        if(v == fa) continue;
        dep[v] = dep[u] + 1, dfs(v, u), size[u] += size[v];
    }
    out[u] = ind;
}

int main()
{
    read(n),read(m);
    for (int i = 1; i <= n; ++i) read(a[i]);
    for (int i = 1, a, b; i < n; ++i) read(a), read(b), add(a, b), add(b, a);
    dep[1] = 1, dfs(1, 0);
    for (int i = 1,x,y,opt; i <= m; ++i) 
    {
        read(opt);
        if(opt == 1) 
        {
            read(x), read(y);
            if (dep[x] & 1) ins(in[x], y), ins(out[x] + 1, -y);
            else ins(in[x], -y), ins(out[x] + 1, y);
        }
        else
        {
            read(x);
            if(dep[x] & 1) printf("%d\n",a[x] + ask(in[x]));
            else printf("%d\n",a[x] - ask(in[x]));
        }
    }
    return 0;
}
```

### [BJOI2018]求和

一棵树，问一条路径上点的深度的 $k$ 次方和。
$k \le 50$，一个很暴力的想法就是维护 $50$ 棵线段树进行大力树剖。
然而空间有些不够，考虑深度的特性。
维护根到节点 $i$ 的深度 $k$ 次方和，然后利用可减性，$f(u) + f(v) - f(lca) - f(fa(lca))$ 即可计算答案。
避免了线段树的巨大空间、常数，可以比较稳妥地通过。

```cpp
#include <cstdio>
#include <algorithm>
#include <ctype.h>
using std::swap;
const int bufSize = 1e6;
#define DEBUG
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
const int mod = 998244353;
const int maxn = 3e5 + 100;
struct node
{
    int to,next;
}E[maxn << 1];
int head[maxn];
inline void addE(const int &x,const int &y)
{
    static int tot = 0;
    E[++tot].next = head[x],E[tot].to = y,head[x] = tot;
}
inline int add(int x,int y)
{
    int res = x + y;
    if (res >= mod) res -= mod;
    return res;
}
inline int mul(int x, int y) { return (1ll * x * y) % mod; }
int n, m, dep[maxn], f[maxn][52], size[maxn], son[maxn], top[maxn],fa[maxn];
void dfs(int u)
{
    int t = dep[u];
    for (int i = 1; i <= 50; ++i) f[u][i] = add(f[fa[u]][i], t), t = mul(t, dep[u]);
    size[u] = 1;
    for (int p = head[u]; p; p = E[p].next) 
    {
        int v = E[p].to;
        if (v == fa[u]) continue;
        fa[v] = u, dep[v] = dep[u] + 1, dfs(v), size[u] += size[v];
        if (size[v] > size[son[u]]) son[u] = v;
    }
}
void dfs2(int u)
{
    if(!son[u]) return;
    top[son[u]] = top[u], dfs2(son[u]);
    for (int p = head[u]; p; p = E[p].next)
    {
        int v = E[p].to;
        if (v == fa[u] || v == son[u]) continue;
        top[v] = v, dfs2(v);
    }
}
int lca(int x, int y)
{
    while(top[x] != top[y])
    {
        if (dep[top[x]] < dep[top[y]]) swap(x, y);
        x = fa[top[x]];
    }
    return dep[x] < dep[y] ? x : y;
}
int main()
{
    read(n);
    for (int i = 1, a, b; i < n; ++i) read(a), read(b), addE(a, b), addE(b, a);
    dfs(1), top[1] = 1, dfs2(1);
    read(m);
    for (int i = 1,a,b,c; i <= m; ++i) 
    {
        read(a), read(b), read(c);
        int l = lca(a,b);
        printf("%d\n", add(add(f[a][c], f[b][c]), add(mod - f[l][c], mod - f[fa[l]][c])));
    }
    return 0;
}
```

### [USACO19DEC]Milk Visits G

[Link](https://www.luogu.com.cn/problem/P5838)

对每个品种维护一棵动态开点的线段树即可。
由于动态开点，空间复杂度是 $O(n \log n)$ 的。

```cpp
#include <cstdio>
#include <algorithm>
#include <ctype.h>
using std::swap;
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
const int maxn = 1e5 + 100;
struct node
{
    int to, next;
} E[maxn << 1];
int head[maxn];
inline void add(const int &x,const int &y)
{
    static int tot = 0;
    E[++tot].next = head[x], E[tot].to = y, head[x] = tot;
}
int n, m, w[maxn], size[maxn], dep[maxn], son[maxn], top[maxn], fa[maxn], dfn[maxn], ind;
int root[maxn], sum[maxn * 30], L[maxn * 30], R[maxn * 30], cnt;
void modify(int l, int r, int& p, int pos)
{
    if(!p) p = ++cnt;
    sum[p]++;
    if (l == r) return;
    int mid = l + r >> 1;
    if (pos <= mid) modify(l, mid, L[p], pos);
    else modify(mid + 1, r, R[p], pos);
}
int ask(int l, int r, int p, int ll, int rr)
{
    if (!p) return 0;
    if (l >= ll && r <= rr) return sum[p];
    int mid = l + r >> 1, ans = 0;
    if (ll <= mid) ans = ask(l, mid, L[p], ll, rr);
    if (rr > mid) ans += ask(mid + 1, r, R[p], ll, rr);
    return ans;
}
void dfs(int u)
{
    size[u] = 1;
    for (int p = head[u]; p; p = E[p].next)
    {
        int v = E[p].to;
        if (v == fa[u]) continue;
        fa[v] = u, dep[v] = dep[u] + 1, dfs(v), size[u] += size[v];
        if (size[v] > size[son[u]]) son[u] = v;
    }
}
void dfs2(int u)
{
    dfn[u] = ++ind;
    if (!son[u]) return;
    top[son[u]] = top[u], dfs2(son[u]);
    for (int p = head[u]; p; p = E[p].next)
    {
        int v = E[p].to;
        if (v == fa[u] || v == son[u]) continue;
        top[v] = v, dfs2(v);
    }
}
bool treeask(int x,int y,int c)
{
    int t = 0;
    while(top[x] != top[y])
    {
        if (dep[top[x]] < dep[top[y]]) swap(x, y);
        t += ask(1, n, root[c], dfn[top[x]], dfn[x]);
        if(t) return 1;
        x = fa[top[x]];
    }
    if (dep[x] > dep[y]) swap(x, y);
    return ask(1, n, root[c], dfn[x], dfn[y]);
}
int main()
{
    read(n), read(m);
    for (int i = 1; i <= n; ++i) read(w[i]);
    for (int i = 1, a, b; i < n; ++i) read(a), read(b), add(a, b), add(b, a);
    dep[1] = 1, dfs(1), top[1] = 1, dfs2(1);
    for (int i = 1; i <= n; ++i) modify(1, n, root[w[i]], dfn[i]);
    for (int i = 1, a, b, c; i <= m; ++i) read(a), read(b), read(c), printf("%d", treeask(a, b, c));
    return 0;
}
```

### [USACO18OPEN]Disruption P

[LInk](https://www.luogu.com.cn/problem/P4374)

显然可以区间取 min 来维护。还可以按边从大到小排序后直接区间赋值。
然而还有更好的做法，贪心排序后，对路径进行类似染色的做法，遇到之前染过色的区间就跳过，随便搞搞即可。
具体题解看 [这篇](https://www.codein.icu/lp4374/)。

### [SHOI2012]魔法树

[Link](https://www.luogu.com.cn/problem/P3833)

显然是树剖的板子，然而还可以用树上差分做到一个 $log$，据说是当年设置的正解。
考虑 $v_i$ 代表根节点到点 $i$ 上的路径，都加上了 $v_i$，那么对路径加权可以看做：
$v_u := v_u + val$
$v_v := v_v + val$
$v_{lca} := v_{lca} - val$
$v_{fa_{lca}} := v_{fa_{lca}} - val$
其实就是常规的树上差分的套路。
那么修改解决了，现在做询问。
有点类似树状数组区间修改区间查询的套路。
对于子树内的某一点 $x$，对答案的贡献为：
$v_x \times (dep_x - dep_u + 1)$
总答案贡献就是：
$\sum v_x \times dep_x - (dep_u - 1) \times \sum v_x$
用两个树状数组维护即可。

```cpp
#include <cstdio>
#include <algorithm>
#include <ctype.h>
const int bufSize = 1e6;
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
const int maxn = 1e5 + 100;
struct node
{
    int to, next;
} E[maxn << 1];
int head[maxn];
inline void addE(const int& x, const int& y)
{
    static int tot = 0;
    E[++tot].next = head[x], E[tot].to = y, head[x] = tot;
}
int n, m, fa[maxn], in[maxn], out[maxn], ind, size[maxn], son[maxn], dep[maxn], top[maxn],id[maxn];
void dfs(int u)
{
    in[u] = ++ind, id[ind] = u, size[u] = 1;
    for (int p = head[u]; p; p = E[p].next)
    {
        int v = E[p].to;
        dep[v] = dep[u] + 1, dfs(v), size[u] += size[v];
        if (size[v] > size[son[u]]) son[u] = v;
    }
    out[u] = ind;
}
void dfs2(int u)
{
    if(!son[u]) return;
    top[son[u]] = top[u],dfs2(son[u]);
    for (int p = head[u]; p; p = E[p].next)
    {
        int v = E[p].to;
        if(v == son[u]) continue;
        top[v] = v, dfs2(v);
    }
}
int lca(int x,int y)
{
    for (; top[x] != top[y]; x = fa[top[x]]) if (dep[top[x]] < dep[top[y]]) std::swap(x, y);
    return dep[x] < dep[y] ? x : y;
}
long long sum1[maxn], sum2[maxn];
inline int lowbit(int x) { return x & -x; }
void add(int x, int k)
{
    int k2 = k * dep[id[x]];
    for (; x <= n; x += lowbit(x)) sum1[x] += k, sum2[x] += k2;
}
long long ask(int x, int k)
{
    if (!x) return 0;
    int u = id[x];
    long long res1 = 0, res2 = 0;
    for (; x > 0; x -= lowbit(x)) res1 += sum1[x], res2 += sum2[x];
    return res2 - (k - 1) * res1;
}
char s[10];
int main()
{
    read(n);
    for (int i = 1, a, b; i < n; ++i) read(a), read(b), ++a, ++b, fa[b] = a, addE(a, b);
    dep[1] = 1, dfs(1), top[1] = 1, dfs2(1);
    read(m);
    for (int i = 1, a, b, c; i <= m; ++i)
    {
        read(s);
        if (s[0] == 'A')
        {
            read(a), read(b), read(c), ++a, ++b;
            int l = lca(a, b);
            add(in[a], c), add(in[b], c), add(in[l], -c);
            if (fa[l]) add(in[fa[l]], -c);
        }
        else read(a), ++a, printf("%lld\n", ask(out[a], dep[a]) - ask(in[a] - 1, dep[a]));
    }
    return 0;
}
```

## 结语

虽然说本来要写轻重链剖分的笔记，但却发现许多树剖习题都有着更为优秀的ad-hoc解法(