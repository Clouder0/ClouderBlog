---
title: "动态DP学习笔记"
date: 2020-10-23T16:10:28+08:00
slug: "ddp"
type: posts
tags:
  - dynamic-programming
  - ddp
  - matrix
  - segment-tree
  - heavydecomposition
categories:
  - algorithm
---

## 前言

动态 DP 问题是猫锟在 WC2018 讲的黑科技，一般用来解决树上的 DP 问题，同时支持点权（边权）修改操作。

在 NOIP2018D2T3 考察后风靡 OI 圈。

一般来说 ，将 DP 的状态转移方程写成矩阵形式，随后通过修改矩阵来实现修改功能。

## 模板例题

[Link](https://www.luogu.com.cn/problem/P4719)

给一棵带点权的树，求最大独立集大小。  
定义 $f(u,0)$ 代表不选 $u$ 点，以 $u$ 为子树中最大权独立集的大小，$f(u,1)$ 为选择 $u$ 点的大小。  

容易得到状态转移方程：

$$f(u,0) = \sum \limits _{v \in son_u} \max(f(v,0),f(v,1))$$  
$$f(u,1) = w_u + \sum \limits _{v \in son_u} f(v,0)$$  
到了修改部分。  
对这棵树进行轻重链剖分，定义 $g(u,0)$ 为 $u$ 子树内不考虑重儿子，$u$ 点不选时的最大权独立集大小。

$$g(u,0) = \sum \limits _{v \in son_u,v \ne heavyson(u)} \max(f(v,0),f(v,1))$$  
同样定义 $g(u,1)$ 为 $u$ 子树内不考虑重儿子，$u$ 点选择时最大权独立集大小。

$$g(u,1) = w_u + \sum \limits _{v \ in son_u,v \ne heavyson(u)} f(v,0)$$  
那么加上重儿子贡献即可得到 $u$ 点状态，有：

$$f(u,0) = g(u,0) + \max(f(heavyson(u),0),f(heavyson(u),1))$$

$$f(u,1) = g(u,1) + f(heavyson(u),0)$$  
可以假定 $g(u,0),g(u,1)$ 已知，稍微改写转移方程：

$$f(u,0) = \max(g(u,0) + f(v,0),g(u,0) + f(v,1))$$

$$f(u,1) = \max(-\infty,g(u,1) + f(v,0))$$

由重儿子状态可以推知当前点状态，并且可以写出转移矩阵：

$$\begin{vmatrix} g(u,0) & g(u,0)\\ g(u,1) & -\infty \end{vmatrix} \times \begin{vmatrix} f(v,0) \\ f(v,1) \end{vmatrix} = \begin{vmatrix}f(u,0) \\ f(u,1)\end{vmatrix}$$

注意这里重定义了矩阵乘法，$C = A \times B,C_{i,k} = \max (A_{i,j} + B_{j,k})$.  
满足结合律，可以用线段树维护一段矩阵相乘的结果，放到树上就是轻重链剖分套线段树。  
可以发现，叶子结点的 $f$ 矩阵就是 $g$ 矩阵的左半部分，考虑如何查询答案。其实就是根节点所在的重链，从叶子节点的矩阵沿着重链一直乘到根节点的矩阵。  
接下来思考修改会造成什么影响，修改 $x$ 点的点权首先会影响 $g(x,1)$  

$$g(u,1) = w_u + \sum \limits _{v \ in son_u,v \ne heavyson(u)} f(v,0)$$  
那么修改该位置即可，然而 $g(x,1)$ 变化后，还会影响到它上方的点。  
与 $x$ 在同一条重链上的点，不用考虑影响，因为线段树的 pushup 已经维护好了。而再往上，就需要修改了。  
考虑 $top[x]$ 是作为 $u$ 点的一个轻儿子来记贡献，现在 $f(top[x])$ 发生了变化，使得 $g(u)$ 也发生了变化，那么就需要修正 $u$ 点处的矩阵。  
一条重链的尾部一定是叶子结点，因此可以记录每个重链的末端，直接利用线段树查询出 $f(top[x])$ 的值，随后对应地进行修改。  
也就是，从 $x$ 点不断跳链跳到根节点。  

然而实现相当多细节，比如说，矩阵乘法满足结合律而不满足交换律。  
如果定义状态矩阵为 $1 \times 2$ 大小，那么就是：

$$(1,2) \times (2,2) = (1,2)$$

那么就需要逆序乘，即线段树右儿子乘上左儿子，才能保证从叶子结点向上推。

然而如果定义 $2 \times 1$ 大小，就有：

$$(2,2) \times (2,1) = (2,1)$$

那么就要先乘上转移矩阵，再乘上叶子结点的状态矩阵，也就是线段树左儿子乘上右儿子。

---
具体看代码吧，也不是很难打。
```cpp
#include <cstdio>
#include <algorithm>
#include <ctype.h>
#include <cstring>
using std::max;
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
int n, m, a[maxn], f[maxn][2];
int fa[maxn], son[maxn], size[maxn], top[maxn], down[maxn], dfn[maxn], id[maxn], ind;
struct matrix
{
    int g[2][2];
    matrix() { std::memset(g, 0, sizeof(g)); }
    matrix operator*(const matrix b) const
    {
        matrix c;
        for (int i = 0; i <= 1; ++i)
            for (int j = 0; j <= 1; ++j)
                for (int k = 0; k <= 1; ++k) c.g[i][k] = max(c.g[i][k], g[i][j] + b.g[j][k]);
        return c;
    }
} A[maxn << 2], G[maxn];
#define ls p << 1
#define rs p << 1 | 1
inline void pushup(int p) { A[p] = A[ls] * A[rs]; }
void build(int l,int r,int p)
{
    if(l ==  r) return (void)(A[p] = G[id[l]]);
    int mid = l + r >> 1;
    build(l, mid, ls), build(mid + 1, r, rs), pushup(p);
}
void modify(int l, int r, int p, int pos)
{
    if (l == r) return (void)(A[p] = G[id[pos]]);  //可以拷贝临时数组，避免在递归中传入结构体常数过大
    int mid = l + r >> 1;
    if (pos <= mid) modify(l, mid, ls, pos);
    else modify(mid + 1, r, rs, pos);
    pushup(p);
}
matrix ask(int l,int r,int p,int ll,int rr)
{
    if(l >= ll && r <= rr) return A[p];
    int mid = l + r >> 1;
    if (ll <= mid && rr > mid) return ask(l, mid, ls, ll, rr) * ask(mid + 1, r, rs, ll, rr);
    else if (ll <= mid) return ask(l, mid, ls, ll, rr);
    else return ask(mid + 1, r, rs, ll, rr);
}
struct node
{
    int to, next;
} E[maxn << 1];
int head[maxn];
inline void add(const int &x,const int &y)
{
    static int tot = 0;
    E[++tot].next = head[x],E[tot].to = y,head[x] = tot;
}
void dfs1(int u)
{
    size[u] = 1, f[u][1] = a[u];
    for (int p = head[u]; p; p = E[p].next)
    {
        int v = E[p].to;
        if(v == fa[u]) continue;
        fa[v] = u, dfs1(v), size[u] += size[v];
        f[u][0] += max(f[v][0], f[v][1]), f[u][1] += f[v][0];
        if (size[v] > size[son[u]]) son[u] = v;
    }
}
void dfs2(int u)
{
    int g0 = 0, g1 = a[u];
    id[dfn[u] = ++ind] = u, down[top[u]] = ind;
    if (son[u]) top[son[u]] = top[u], dfs2(son[u]);
    for (int p = head[u]; p; p = E[p].next)
    {
        int v = E[p].to;
        if (v == son[u] || v == fa[u]) continue;
        top[v] = v, dfs2(v), g0 += max(f[v][0], f[v][1]), g1 += f[v][0];
    }
    G[u].g[0][0] = G[u].g[0][1] = g0;
    G[u].g[1][0] = g1, G[u].g[1][1] = -(1 << 30);
}
void update(int x,int val)
{
    G[x].g[1][0] += val - a[x], a[x] = val;
    while(x)
    {
        matrix old = ask(1, n, 1, dfn[top[x]], down[top[x]]);//计算出旧的 f[v]
        modify(1, n, 1, dfn[x]);
        matrix now = ask(1, n, 1, dfn[top[x]], down[top[x]]);//计算出新的 f[v]
        x = fa[top[x]];
        G[x].g[0][0] += max(now.g[0][0], now.g[1][0]) - max(old.g[0][0], old.g[1][0]);
        G[x].g[0][1] = G[x].g[0][0];
        G[x].g[1][0] += now.g[0][0] - old.g[0][0];
    }
}
int main()
{
    read(n), read(m);
    for (int i = 1; i <= n; ++i) read(a[i]);
    for (int i = 1, u, v; i < n; ++i) read(u), read(v), add(u, v), add(v, u);
    dfs1(1), top[1] = 1, dfs2(1), build(1, n, 1);
    for (int i = 1, x, val; i <= m; ++i)
    {
        read(x), read(val), update(x, val);
        matrix res = ask(1, n, 1, 1, down[1]);
        printf("%d\n",max(res.g[0][0],res.g[1][0]));
    }
    return 0;
}
```

## 保卫王国

一道看起来很像板子题的东西。  
最小覆盖集 = 全集 - 最大独立集  
尽管可以利用这个转化为板子，但直接求最小覆盖集也是可行的。  
定义 $f(u,0)$ 为 $u$ 为根的子树内，不选 $u$ 的最小代价，$f(u,1)$ 为选 $u$ 的最小代价。

$$f(u,0) = \sum f(v,1)$$  
$$f(u,1) = \sum \min(f(v,0),f(v,1))$$  
同样按照套路，轻重链剖分，定义 $g(u,0)$ 为除去重儿子外，不选 $u$ 的最小代价，$g(u,1)$ 为除去重儿子外，选 $u$ 的最小代价。

$$g(u,0) = \sum \limits _{v \ne son(u)} f(v,1)$$  
$$g(u,1) = w_u + \sum \limits _{v \ne son(u)} \min(f(v,0),f(v,1))$$  
改写状态转移方程，有：

$$f(u,0) = f(son,1) + g(u,0)$$

$$f(u,1) = \min(f(son,0) + g(u,1),f(son,1) + g(u,1))$$

接下来开始构造矩阵，按照线段树左儿子乘上右儿子的习惯，构造 $2 \times 1$ 的状态矩阵与 $2 \times 2$ 的转移矩阵。

$$\begin{vmatrix} +\infty & g(u,0) \\ g(u,1) & g(u,1) \end{vmatrix} \times \begin{vmatrix} f(son,0) \\ f(son,1) \end{vmatrix} = \begin{vmatrix} f(u,0) \\ f(u,1) \end{vmatrix}$$  
接下来按照套路，思考如何在修改时维护 $fa[top[x]]$ 的 $g$ 矩阵。

$$g(u,0) = \sum \limits _{v \ne son(u)} f(v,1)$$

$$g(u,1) = w_u + \sum \limits _{v \ne son(u)} \min(f(v,0),f(v,1))$$

简单的加加减减，没太大区别。  
强制选点，强制不选点，只要修改转移矩阵即可。  
如果强制选点，就让不选点的转移矩阵对应位置变成 $+\infty$，如果强制不选点，就让选点的对应位置变成 $+\infty$  
为了方便撤销，可以不直接修改，而使用加上 $+\infty$ 的方法，撤销时减去 $+\infty$ 即可。

然而需要注意的是，此时的 $f$ 矩阵与 $g$ 矩阵形态不同了，需要特判叶子结点，并将 $f$ 矩阵塞到对应位置。  
修改的时候，也同样需要特判叶子结点。  
似乎也有规避这个特判的方法，就是改改矩阵形态，让它们恰好对正。不过不知道行不行，我直接硬上特判了。

```cpp
#include <cstdio>
#include <algorithm>
#include <ctype.h>
#include <cstring>
const int bufSize = 1e6;
using std::min;
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
inline void add(const int &x,const int &y)
{
    static int tot = 0;
    E[++tot].next = head[x],E[tot].to = y,head[x] = tot;
}
int n, m, w[maxn];
char types[10];
int size[maxn], fa[maxn], son[maxn], top[maxn], dfn[maxn], id[maxn], down[maxn], ind;
long long f[maxn][2];
const long long inf = 1ll << 40;
struct matrix
{
    long long g[2][2];
    matrix(){std::memset(g,0x3f,sizeof(g));}
    matrix operator*(const matrix b) const
    {
        matrix c;
        for (int i = 0; i < 2; ++i)
            for (int j = 0; j < 2; ++j)
                for (int k = 0; k < 2; ++k) c.g[i][k] = min(c.g[i][k], g[i][j] + b.g[j][k]);
        return c;
    }
} A[maxn << 2], G[maxn];
#define ls p << 1
#define rs p << 1 | 1
inline void pushup(int p) { A[p] = A[ls] * A[rs]; }
void build(int l,int r,int p)
{
    if(l == r) return (void)(A[p] = G[id[l]]);
    int mid = l + r >> 1;
    build(l, mid, ls), build(mid + 1, r, rs), pushup(p);
}
void modify(int l,int r,int p,int pos)
{
    if (l == r) return (void)(A[p] = G[id[l]]);
    int mid = l + r >> 1;
    if (pos <= mid) modify(l, mid, ls, pos);
    else modify(mid + 1, r, rs, pos);
    pushup(p);
}
matrix ask(int l,int r,int p,int ll,int rr)
{
    if (l >= ll && r <= rr) return A[p];
    int mid = l + r >> 1;
    if (ll <= mid && rr > mid) return ask(l, mid, ls, ll, rr) * ask(mid + 1, r, rs, ll, rr);
    if (ll <= mid) return ask(l, mid, ls, ll, rr);
    return ask(mid + 1, r, rs, ll, rr);
}
void dfs1(int u)
{
    size[u] = 1, f[u][1] = w[u];
    for (int p = head[u]; p; p = E[p].next)
    {
        int v = E[p].to;
        if(v == fa[u]) continue;
        fa[v] = u, dfs1(v), size[u] += size[v];
        f[u][0] += f[v][1], f[u][1] += min(f[v][0], f[v][1]);
        if (size[v] > size[son[u]]) son[u] = v;
    }
}
void dfs2(int u)
{
    id[dfn[u] = ++ind] = u,down[top[u]] = ind;
    if (son[u]) top[son[u]] = top[u], dfs2(son[u]);
    else
    {
        //特判叶子
        G[u].g[0][0] = 0,G[u].g[1][0] = w[u];
        return;
    }
    G[u].g[0][0] = inf, G[u].g[0][1] = 0, G[u].g[1][0] = w[u];
    for (int p = head[u]; p; p = E[p].next)
    {
        int v = E[p].to;
        if (v == son[u] || v == fa[u]) continue;
        top[v] = v, dfs2(v), G[u].g[0][1] += f[v][1], G[u].g[1][0] += min(f[v][0], f[v][1]);
    }
    G[u].g[1][1] = G[u].g[1][0];
}
void treeupdate(int x)
{
    while(x)
    {
        matrix old = ask(1, n, 1, dfn[top[x]], down[top[x]]);
        modify(1, n, 1, dfn[x]);
        matrix now = ask(1, n, 1, dfn[top[x]], down[top[x]]);
        x = fa[top[x]];
        G[x].g[0][1] += now.g[1][0] - old.g[1][0];
        G[x].g[1][0] += min(now.g[0][0], now.g[1][0]) - min(old.g[0][0], old.g[1][0]);
        G[x].g[1][1] = G[x].g[1][0];
    }
}
inline void change(int a, int x, int val)
{
    if(son[a])
    {
        if (x) G[a].g[0][1] += val * inf;
        else G[a].g[1][0] += val * inf, G[a].g[1][1] += val * inf;
    }
    else
    {
        //叶子结点，修改f矩阵
        if (x) G[a].g[0][0] += val * inf;
        else G[a].g[1][0] += val * inf;
    }
}
long long update(int a, int x, int b, int y)
{
    change(a, x, 1), change(b, y, 1);
    treeupdate(a), treeupdate(b);
    matrix t = ask(1, n, 1, 1, down[1]);
    long long res = min(t.g[0][0], t.g[1][0]);
    change(a, x, -1), change(b, y, -1);
    treeupdate(a), treeupdate(b);
    return res;
}
int main()
{
    read(n), read(m), read(types);
    for (int i = 1; i <= n; ++i) read(w[i]);
    for (int i = 1, u, v; i < n; ++i) read(u), read(v), add(u, v), add(v, u);
    dfs1(1), top[1] = 1, dfs2(1), build(1, n, 1);
    for (int i = 1, a, x, b, y; i <= m; ++i)
    {
        read(a), read(x), read(b), read(y);
        long long res = update(a, x, b, y);
        if (res >= inf) puts("-1");
        else printf("%lld\n", res);
    }
    return 0;
}
```