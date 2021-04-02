---
title: '树上点与路径的问题的在线差分解法'
date: 2020-10-27T16:13:02+08:00
slug: "treepath"
type: posts
tags:
  - tree
  - fenwick-tree
categories:
  - Algorithm
---

## 前言

- 给出一棵树，在线给出若干条路径，在线询问某个点被多少条路径覆盖。
- 给出一棵树，在线给出若干个关键点，在线询问某条路径上覆盖了多少个关键点。

这两种问题，有着显而易见的轻重链剖分解法，甚至可以说是轻重链剖分的经典应用。  
然而，在某些时候，我们认为轻重链剖分 $O(\log^2 n)$ 的代价太大了。  
那么可以使用树上差分的思想，利用树状数组维护。  

## 点被路径覆盖

给出路径，询问某个点被多少条路径覆盖。  
考虑路径先全部给出怎么做。  
一个显然的做法是进行树上差分，将 $u \to v$ 的路径拆分成 $u \to lca$ 与 $v \to lca$，然后对 $fa(lca)$ 打 $-1$ 标记，对 $lca$ 打 $-1$ 标记，对 $u$ 和 $v$ 打 $1$ 标记即可。  
随后对全树进行一次搜索即可求出每个点被覆盖多少次。

那么在线如何做呢？  
求出每个点的 dfs 序，然后使用树状数组维护单点修改、区间查询。  
对于每条路径，按照树上差分的方法进行单点修改。  
查询 $u$ 点被多少条路径覆盖，就查询 $u$ 的子树内的权值和。  
可以这么理解：在树上差分的过程中，$u$ 的子树内的标记，都能在搜索过程中转移到 $u$ 点，而相当于直接求出子树内的标记和来得到该点的值。  

例题：[\[HNOI2016\]网络](https://www.luogu.com.cn/problem/P3250)  

关键代码：

```cpp
if(Q[i].type == 2)
{
    int num = Bit::ask(dfn[Q[i].u] + size[Q[i].u] - 1) - Bit::ask(dfn[Q[i].u] - 1);
    if (num == edgenum) q1[++cnt1] = Q[i];
    else q2[++cnt2] = Q[i];
}
else
{
    int t = Q[i].type == 0 ? 1 : -1;
    if(Q[i].val > H[mid])
    {
         Bit::add(dfn[Q[i].u], t), Bit::add(dfn[Q[i].v], t), Bit::add(dfn[Q[i].l], -t);
         if (fa[Q[i].l]) Bit::add(dfn[fa[Q[i].l]], -t);
         q2[++cnt2] = Q[i], edgenum += t;
    }
    else q1[++cnt1] = Q[i];
}
```

## 路径覆盖点

给出关键点，询问某条路径覆盖了多少个关键点。

考虑差分怎么做，预处理出 $f(i)$ 代表每个点到根节点的路径上有多少条关键点，随后对于路径 $u \to v$，可以拆成 $u \to lca$ 与 $v \to lca$，那么答案就是 $f(u) + f(v) - f(lca) - f(fa(lca))$.

那么加上在线，一个关键点会对其子树内的点的 $f$ 做出贡献。  
维护区间修改、单点查询即可。

例题：[\[CTSC2008\] 网络管理](https://www.luogu.com.cn/problem/P4175)

关键代码：

```cpp
if(Q[i].type <= 2)
{
    if (Q[i].val > H[mid])
    {
        int t = Q[i].type == 1 ? 1 : -1;
        Bit::addrange(dfn[Q[i].u], dfn[Q[i].u] + size[Q[i].u] - 1, Q[i].type == 1 ? 1 : -1);
        if(tvis[Q[i].u] != tim) tsum[Q[i].u] = 0,tvis[Q[i].u] = tim;
        tsum[Q[i].u] += t, q2[++cnt2] = Q[i];
    }
    else q1[++cnt1] = Q[i];
}
else
{
    if (tvis[Q[i].l] != tim) tsum[Q[i].l] = 0, tvis[Q[i].l] = tim;
    int num = Bit::ask(dfn[Q[i].u]) + Bit::ask(dfn[Q[i].v]) - Bit::ask(dfn[Q[i].l]) * 2 + tsum[Q[i].l];
    if (num >= Q[i].val) q2[++cnt2] = Q[i];
    else Q[i].val -= num, q1[++cnt1] = Q[i];
}
```