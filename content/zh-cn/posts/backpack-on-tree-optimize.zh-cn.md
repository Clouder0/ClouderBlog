---
title: "树上背包的优化"
date: 2020-04-12T22:00:53+08:00
slug: "backpack-on-tree-optimize"
type: posts
tags:
  - backpack-on-tree
  - optimize
categories:
  - algorithm
---

## 加入

本文为笔者年少无知时所作，大体内容应该问题不大，但可能夸大了运用指针带来的优化效果。

请各位读者批判地阅读吧……


## 正文

[例题链接](https://www.luogu.com.cn/problem/U53204)

本文旨在介绍树上背包的优化。

可见例题，例题中 $N,M \in [1,100000]$ 的数据量让 $O(nm^2)$ 的朴素树上背包 T 到飞起，我们需要考虑优化。

个人会将各种优化讲到极限（当然是本蒟蒻的极限）。

根据一番学习，我也认为上下界优化最简单易理解……

上下界优化这位神犇的博客相当不错了：[戳我 % 他](https://www.cnblogs.com/ouuan/p/BackpackOnTree.html)

我也口胡两句吧。

普通做法：

```cpp
for (j=m+1;j>=1;--j)//枚举背包容量
	for (k=1;k<j;++k)//枚举在子树中选择多少
		f[u][j]=max(f[u][j],f[u][k]+f[v][j-k]);
```

那么 size 优化非常简单好想：

```cpp
for (j=min(m+1,size[u]);j>=1;--j)//枚举背包容量
	for (k=1;k<j&&k<=size[v];++k)//枚举在子树中选择多少
		f[u][j]=max(f[u][j],f[u][k]+f[v][j-k]);
```

道理也很简单，选完就那么多，肯定不能枚举到超过的。

于是能 AC 这道题，用时 17s。

但再想想，我们选择的$k$的下界其实也是会被约束的。

因为选到$j$的总容量的时候，假定前面的全部取完，$k$都必须要到达一个值才能满足条件。

例子：

$size[u] = size[son1] + size[son2] + size[son3]$

我们枚举时，比如 $j = size[son1] + size[son2] + a$ 的情况，

我们至少要在 $son3$ 中取 $a$ 个节点才能达到此容量。

因此就能得到上下界优化：

```cpp
void dfs(int u)
{
    siz[u]=1;
    f[u][1]=a[u];
    int i,j,k,v;
    for (i=head[u];i;i=nxt[i])
    {
        v=to[i];
        dfs(v);
        for (j=min(m+1,siz[u]+siz[v]);j>=2;--j)//这里做了小改动，因为1的更新肯定没有意义
            for (k=max(1,j-siz[u]);k<=siz[v]&&k<j;++k)
                f[u][j]=max(f[u][j],f[u][j-k]+f[v][k]);
        siz[u]+=siz[v];
    }
}
```

这里对 size 数组的更新做了特殊处理，可以更方便地得到前面所有子树的节点数总和。
于是更进一步，达到了 12s 的成绩。

那么还能不能更快呢？其实是可以的。

我们发现内层循环需要 2 个判断语句，有什么办法缩成一个？
当然可以开临时变量来存，但我们甚至可以换一种 dp 方式！（思路来源于某位神犇，他的代码用了刷表法无师自通地进行了 $O(nm)$ 优化导致过去“指点”的我转为“%%%”状态）

## 刷表法

刷表法怎么写呢？其实也很简单：

```cpp
void dfs(int u)
{
    siz[u]=1;
    f[u][1]=a[u];
    int i,j,k,v;
    for (i=head[u];i;i=nxt[i])
    {
        v=to[i];
        dfs(v);
        for (j=min(m,siz[u]);j>=1;--j)//在之前子树&&根中选择的节点数，这里要取1是因为肯定要取根节点
            for (k=1;k<=siz[v]&&j+k<=m+1;++k)//在当前子树取得节点数
                f[u][j+k]=max(f[u][j+k],f[u][j]+f[v][k]);
        siz[u]+=siz[v];
    }
}
```

这时候，我们就可以将内层循环的两个判断语句合为一个了：

```cpp
void dfs(int now)
{
    size[now] = 1;
    f[now][1] = w[now];
    int v;
    for (int p = head[now]; p; p = lines[p].next)
    {
        v = lines[p].to;
        dfs(v);
        for (int j = min(size[now], m); j; --j)
            for (int k = min(size[v], m + 1 - j); k; --k)
                f[now][j + k] = max(f[now][j + k], f[now][j] + f[v][k]);
        size[now] += size[v];
    }
}
```

省去了一个判断，对常数的优化还是不可小觑的。


## 下标映射

对于例题，由于 $n,m$ 过大，开二维肯定开不下，肯定要扁平化为一维。

因为有一个超级源点，因此背包最大容量其实为 $m+1$，而 $[0,m+1]$ 间有 $m+2$ 个位置。

故有：

```cpp
inline int pos(const int &x,const int &y)
{
	return x * (m+2) + y;//注意此处x可能为0
}
```

但是事实上，每次都计算这个 pos 带来了大量的计算。

多大量呢？

当初用填表法时，我将这个函数换成了 define，总时间从 12s 提升到了 8s。

显然因为这个 pos 反复计算，消耗了大量的时间。

那么是否还有比宏定义更优的方法呢？我翻了翻最优解，除了题目作者本人在调整数据规模时的弱数据 AC 外，第一位是一位名为 *WarlockAkk* 的神犇，用时仅 4.2s！

这究竟是何等黑魔法？我点开源码开始膜拜，于是看到：

```cpp
bfo(i,0,n+1){
        d[i]=spa+idx;
		idx+=m+2;
	}
```

这是什么意思呢？$d$ 是一个$int^*$的数组，于是我恍然大悟：

> 可以预处理出一个映射数组，将二维的对映射数组的访问映射到一维的保存数组中。

>

具体实现方式：

```cpp
int dp[100001000];
int *f[MAXN]; //f[i][j] points to the dp arr.
int k, pointer = 0;
    f[0] = &dp[0]; //special
    for (int i = 1; i <= n; ++i)
    {
        pointer += m + 2;
        f[i] = &dp[pointer]; //special
    }
```

我们将这两行代码插入到读入的循环中，就可以得到映射数组 $f$，我们就能直接用 $f[i][j]$ 来访问了！

并且因为 $f[i]$ 存的索引直接加上 $j$ 就能得到地址，我们实际上避免了两个大数的乘法，而使其变成了加法。

举例：

原先访问方式：

$dp[x * (m+2) + y ]$ 进行了一次乘法一次加法

解析一下就是：

```cpp
return dp + (x * (m+2) + y);
```

而现在的访问方式：

$(f[x] + y)$

解析一下就是：

```cpp
return (f + x) + y;
```

效率提升相当显著。

同时注意我们的预处理方式：

```cpp
int pointer = 0;
pointer += m + 2;
```

写成加法的形式，与乘法形式对比：

```cpp
pointer = (m + 2) * i;
```

效率如何很显然了。

那么下标映射后到底有多快呢？

有多快呢？

我们看结论吧。


## 总结

| 填表法 | 填表法 with O(2) | 刷表法 | 刷表法 with O(2) | 下标映射 + 刷表法 with O(2) |
| ------ | ---------------- | ------ | ---------------- | --------------------------- |
| $8s$     | $7.5s$             | $7s$     | $6.5s$             | $2.4s$                        |

可以发现，吸氧对于这种情况提升不明显。

而下标映射 **快、极快、巨快！**

![最优解](https://img-blog.csdnimg.cn/20191212221209892.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01pc3NpbmdfQ2xvdWQ=,size_16,color_FFFFFF,t_70)

因此在卡常优化时我们可以多想想使用指针等玄学进行优化，往往会有意想不到的提升。

如 lower_bound 等函数直接使用迭代器等……

That's all.

## Code

```cpp
#pragma GCC target("avx")
#pragma GCC optimize(2)
#pragma GCC optimize(3)
#pragma GCC optimize("Ofast")
#pragma GCC optimize("inline")
#pragma GCC optimize("-fgcse")
#pragma GCC optimize("-fgcse-lm")
#pragma GCC optimize("-fipa-sra")
#pragma GCC optimize("-ftree-pre")
#pragma GCC optimize("-ftree-vrp")
#pragma GCC optimize("-fpeephole2")
#pragma GCC optimize("-ffast-math")
#pragma GCC optimize("-fsched-spec")
#pragma GCC optimize("unroll-loops")
#pragma GCC optimize("-falign-jumps")
#pragma GCC optimize("-falign-loops")
#pragma GCC optimize("-falign-labels")
#pragma GCC optimize("-fdevirtualize")
#pragma GCC optimize("-fcaller-saves")
#pragma GCC optimize("-fcrossjumping")
#pragma GCC optimize("-fthread-jumps")
#pragma GCC optimize("-funroll-loops")
#pragma GCC optimize("-fwhole-program")
#pragma GCC optimize("-freorder-blocks")
#pragma GCC optimize("-fschedule-insns")
#pragma GCC optimize("inline-functions")
#pragma GCC optimize("-ftree-tail-merge")
#pragma GCC optimize("-fschedule-insns2")
#pragma GCC optimize("-fstrict-aliasing")
#pragma GCC optimize("-fstrict-overflow")
#pragma GCC optimize("-falign-functions")
#pragma GCC optimize("-fcse-skip-blocks")
#pragma GCC optimize("-fcse-follow-jumps")
#pragma GCC optimize("-fsched-interblock")
#pragma GCC optimize("-fpartial-inlining")
#pragma GCC optimize("no-stack-protector")
#pragma GCC optimize("-freorder-functions")
#pragma GCC optimize("-findirect-inlining")
#pragma GCC optimize("-fhoist-adjacent-loads")
#pragma GCC optimize("-frerun-cse-after-loop")
#pragma GCC optimize("inline-small-functions")
#pragma GCC optimize("-finline-small-functions")
#pragma GCC optimize("-ftree-switch-conversion")
#pragma GCC optimize("-foptimize-sibling-calls")
#pragma GCC optimize("-fexpensive-optimizations")
#pragma GCC optimize("-funsafe-loop-optimizations")
#pragma GCC optimize("inline-functions-called-once")
#pragma GCC optimize("-fdelete-null-pointer-checks")
#include <cstdio>
using namespace std;
const int MAXN = 100100;
inline int max(const int &a, const int &b) { return a > b ? a : b; }
inline int min(const int &a, const int &b) { return a < b ? a : b; }
char buf[100000], *p1 = buf, *p2 = buf;
#define nc() p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, 100000, stdin), p1 == p2) ? EOF : *p1++
template <typename T>
inline void read(T &r)
{
    static char c;r = 0;
    for (c = nc(); c > '9' || c < '0'; c = nc());
    for (; c >= '0' && c <= '9'; r = (r << 1) + (r << 3) + (c ^ 48), c = nc());
}
struct node
{
    int to, next;
    node() {}
    node(const int &_to, const int &_next) : to(_to), next(_next) {}
} lines[MAXN];
int head[MAXN];
void add(const int &x, const int &y)
{
    static int tot = 0;
    lines[++tot] = node(y, head[x]), head[x] = tot;
}
int n, m;
int dp[100001000];
int *f[MAXN]; //f[i][j] points to the dp arr.
int size[MAXN], w[MAXN];
void dfs(int now)
{
    int v;
    size[now] = 1;
    f[now][1] = w[now];
    for (int p = head[now]; p; p = lines[p].next)
    {
        v = lines[p].to;
        dfs(v);
        for (int i = min(size[now], m); i; --i)
            for (int j = min(size[v], m + 1 - i); j; --j)
                f[now][i + j] = max(f[now][i + j], f[now][i] + f[v][j]);
        size[now] += size[v];
    }
}
int main()
{
    read(n);
    read(m);
    int k, pointer = 0;
    f[0] = &dp[0]; //special
    for (int i = 1; i <= n; ++i)
    {
        pointer += m + 2;
        f[i] = &dp[pointer]; //special
        read(k);
        add(k, i); //we can set the point(0) into a vitual node,which is the root of the tree
        read(w[i]);
    }
    dfs(0);
    printf("%d", f[0][m + 1]);
    return 0;
}
```
