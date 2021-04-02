---
title: "哈希表"
date: 2020-05-08T12:52:20+08:00
slug: "hashtable"
type: posts
tags:
  - hash
categories:
  - Algorithm
---

# 哈希表

哈希表，又称散列表，是一种储存键值对的数据结构。

哈希表的基础思想是拿空间换时间，哈希表的期望复杂度是 $O(1)$ 的。

一般来说，对于某 key 值，哈希后得到对应的下标，代表其在哈希表中的位置。

$\texttt{hashtable}[\operatorname{hash}(key)] = \texttt{value}$

哈希表的核心就在于 $\operatorname{hash}$ 函数。

## 哈希冲突

哈希冲突是哈希表极力避免的情况。由于对 key 值进行哈希，不能保证不同 key 值的哈希一定不同，因此可能出现 $x_1 \neq x_2$ ，使得 $\operatorname{hash}(x_1) = \operatorname{hash}(x_2)$ ，即出现哈希冲突。

如果不考虑哈希冲突，就会出现误判的情况。而要解决哈希冲突，往往会使哈希表复杂度退化。

不同的实现方法，本质上就是用不同方法避免哈希冲突。

## 桶

可以将桶看做一种特殊的哈希表，存储整数型的键值对。

$\operatorname{hash}(x) = x$

这样能保证在值域范围内不冲突，key 值与其哈希值一一对应，在值域不大时是一种良好的数据结构。

```cpp
inline int hash(int x){return x;}
bool ht[maxn];
inline bool find(int x){return ht[hash(x)];}
inline void insert(int x){ht[hash(x)] = true;}
```

## 单模数哈希表

单模数哈希表是使用广泛、代码简单的一种实现方式。

在值域范围较大，无法承受其空间时，可以对值取模来减少使用空间。

```cpp
const int mod = 10000103;//usually a prime
bool ht[mod + 100];
inline int hash(int x){return ((x % mod) + mod) % mod;}
inline bool find(int x){return ht[hash(x)];}
inline void insert(int x){ht[hash(x)] = true;}
```

上述代码就是一种简单的实现。

然而这种实现没有考虑哈希冲突，在处理 $x$ 与 $x + mod$ 时就会发生明显的错误。

## 线性探测法

为了解决单模数哈希表的哈希冲突，有线性探测法。

具体地，当 $key[\operatorname{hash}(x)] \neq x$ 时，检查 $\operatorname{hash}(x) + 1,\operatorname{hash}(x) + 2,\ldots$ 位置的 $key$ 值是否与 $x$ 相等，若相等则读取 $val$ 值。

在最劣情况下，单次操作是 $O(n)$ 复杂度的。

```cpp
struct LinearAttemptHashTable
{
    static const int mod = 10000103;
    int keytable[mod + 100], valtable[mod + 100];
    LinearAttemptHashTable() { memset(keytable, 0, sizeof(keytable)), memset(valtable, 0, sizeof(valtable)); }
    int hash(int x) { return ((x % mod) + mod) % mod; }
    int & operator[](int key)
    {
        int p = hash(key);
        while (keytable[p] && keytable[p] != key) p = (p + 1) % mod;
        keytable[p] = key;
        return valtable[p];
    }
    int* find(int key)
    {
        int p = hash(key);
        while (keytable[p] && keytable[p] != key) p = (p + 1) % mod;
        if(keytable[p] == key) return &valtable[p];
        return NULL;
    }
};
```

## 拉链法

拉链法是另一种解决哈希冲突的方法。

在 $\operatorname{hash}(key)$ 冲突时，在 $ht[key]$ 处创建链表，将元素加入链表即可。

在查询时，遍历对应位置的链表。

最劣情况下，单次操作是 $O(n)$ 复杂度的。

然而大部分情况下表现良好。注意需要开链表的话，可以将模数适当调小。

或者用类似建图时的写法，开固定大小的内存池。

```cpp
struct ListHashTable
{
    static const int mod = 100103;
    std::list<std::pair<int, int> > valtable[mod + 100];
    int hash(int x)
    {
        int res = x % mod;
        if (res < 0) res += mod;
        return res;
    }
    int & operator[](int key)
    {
        int p = hash(key);
        for (std::list<std::pair<int, int> >::iterator it = valtable[p].begin(); it != valtable[p].end(); ++it)
            if (it->first == key) return it->second;
        valtable[p].push_front(std::make_pair(key, 0));
        return valtable[p].begin()->second;
    }
    int* find(int key)
    {
        int p = hash(key);
        for (std::list<std::pair<int, int> >::iterator it = valtable[p].begin(); it != valtable[p].end(); ++it)
            if (it->first == key) return &it->second;
        return NULL;
    }
};
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
};
```

## 重哈希法

该方法名称为本人杜撰的。

上面两种方法虽然简单，而随机数据下也称得上有效，但由于广为人知，容易遇到精心构造的数据。

而这种方法不那么常见，而由于随机性较大，也不那么好卡。

当 $p = \operatorname{hash}(key)$ 位置已占用时， $p := \operatorname{hash}(p)$ ，即重哈希。

而这里的哈希函数发挥空间很大，只要保证 $\operatorname{hash}(\operatorname{hash}(x)) \neq \operatorname{hash}(x)$ 并且让 $\operatorname{hash}(x)^C = \operatorname{hash}(x)$ 的 $C$ 尽量大即可。

可以说，线性探测法是重哈希法的一种特殊实现。

```cpp
struct AttemptHashTable
{
    static const int mod = 10000103;
    int keytable[mod + 100], valtable[mod + 100];
    AttemptHashTable() { memset(keytable, 0, sizeof(keytable)), memset(valtable, 0, sizeof(valtable)); }
    int hash(int x) 
    { 
        int res = (x + 195781) % mod;
        if(res < 0) res += mod;
        return res;
    }
    int & operator[](int key)
    {
        int p = hash(key);
        while (keytable[p] && keytable[p] != key) p = hash(p);
        keytable[p] = key;
        return valtable[p];
    }
    int* find(int key)
    {
        int p = hash(key);
        while (keytable[p] && keytable[p] != key) p = hash(p);
        if(keytable[p] == key) return &valtable[p];
        return NULL;
    }
};
```

其中哈希函数 $\operatorname{hash}(x) = x + C$ ， $C$ 是质数时，哈希表发挥较好。

$x$ 可以存储到的范围为 $y = x + C \times k_1 + mod \times k_2$ ，其中 $k_1,k_2$ 为整数。

$C \times k_1 + mod \times k_2 = \gcd(C,mod)$ 有解，而 $\gcd(C,mod) = 1$ ，则 $y$ 可以取到整个数组。

## 多模数哈希

多模数哈希也是一种简单的哈希方法。

当 $\pmod {p_1}$ 意义下有冲突，则更换模数，求出 $\pmod {p_2}$ 意义下的下标。

模数可以使用随机数，能防止被卡。

值得注意的是，在模数较少的情况下，仍可能出现错误。即多个模数意义下的下标都被占用的情况。

可以直接取很多个模数，尽量减小这种可能性。

# 结语

哈希表的实现千千万万种，最为常用的线性探测法、拉链法在实际应用中都有不错的表现。

以上仅为几种广为人知的、较为简单的哈希表的实现，供各位读者参考。