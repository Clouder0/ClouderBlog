---  
title: '[Atcoder][CF]简单题选做练习笔记 2'  
date: 2020-11-17T10:06:09+08:00  
slug: "atcftrain2"  
type: posts  
tags:  
  - codeforces  
  - atcoder  
categories:  
  - Algorithm  
---  
  
## 前言  
  
接着上篇继续刷。文章太长了被迫拆分。  
  
### AT5759 ThREE  
  
构造题，我最讨厌的题型之一。  
  
$$ p_i \times p_j \equiv 0 \pmod 3 $$  
  
或者：  
  
$$ p_i + p_j \equiv 0 \pmod 3 $$  
  
第一个式子，要求 $p_i \equiv 0 \pmod 3$，第二个式子要求 $p_i \equiv 1 \pmod 3$ 且 $p_j \equiv 2 \pmod 3$ 或者反过来。  
  
一个简单的思路就是按 $i \bmod 3$ 进行分类，然后考虑怎么做。  
  
然后发现这个东西特别难做，因为距离为 $3$ 还会出现跨过祖先的点对。  
  
但是有一点是确定的，即距离为 $3$ 的点对深度奇偶性一定不同。虽然奇偶性不同的点不一定距离为 $3$，但如果保证奇偶性不同的点都能满足，那么距离为 $3$ 当然也满足。  
  
于是可以按深度分类，分为 $X$ 与 $Y$，即深度为奇与偶的集合。  
  
需要按数量进行讨论。  
  
$|X|,|Y| > \lfloor \dfrac{n}{3} \rfloor$  
  
两个都比较大的时候，将 $p_i \equiv 1 \pmod 3$ 与 $p_i \equiv 2 \pmod 3$ 分别放入，再把 $p_i \equiv 0 \pmod 3$ 的塞入剩下的位置。这样对于所有的奇偶性不同的点对，都有 $p_i \equiv 0 \pmod 3$ 或者 $p_i \equiv 1 \pmod 3$ 且 $p_j \equiv 2 \pmod 3$，因此可以满足。  
  
$|X| > \lfloor \dfrac{n}{3} \rfloor, |Y| \le \lfloor \dfrac{n}{3} \rfloor$  
  
如果出现这种情况，说明 $|Y|$ 特别小，而要保证 $X$ 与 $Y$ 中任意两组都满足的话，可以直接把 $p_i \equiv 0 \pmod 3$ 这个万金油塞到 $Y$ 中，然后 $X$ 随便放就行了。  
  
$|X| < \lfloor \dfrac{n}{3} \rfloor$ 同理。  
  
由于 $|X| + |Y| = n$，因此不会存在其他的情况了。  
  
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
    int to, next;  
} E[maxn << 1];  
int head[maxn];  
inline void add(const int& x, const int& y)  
{  
    static int tot = 0;  
    E[++tot].next = head[x], E[tot].to = y, head[x] = tot;  
}  
int n, col[maxn], cnt[2], res[maxn];  
void dfs(int u, int fa)  
{  
    cnt[col[u]]++;  
    for (int p = head[u]; p; p = E[p].next)  
    {  
        int v = E[p].to;  
        if(v == fa) continue;  
        col[v] = col[u] ^ 1, dfs(v, u);  
    }  
}  
int main()  
{  
    read(n);  
    for (int i = 1, a, b; i < n; ++i) read(a), read(b), add(a, b), add(b, a);  
    dfs(1, 1);  
    int mod1 = 1, mod2 = 2, mod0 = 3;  
    if(cnt[0] > n / 3 && cnt[1] > n / 3)  
    {  
        for (int i = 1; i <= n; ++i)   
        {  
            if (col[i] == 0)  
            {  
                if (mod1 <= n) res[i] = mod1, mod1 += 3;  
                else res[i] = mod0, mod0 += 3;  
            }  
            else  
            {  
                if (mod2 <= n) res[i] = mod2, mod2 += 3;  
                else res[i] = mod0, mod0 += 3;  
            }  
        }  
    }  
    else if(cnt[0] <= n / 3)  
    {  
        for (int i = 1; i <= n; ++i)   
        {  
            if (col[i] == 0)  
            {  
                if (mod0 <= n) res[i] = mod0, mod0 += 3;  
            }  
            else  
            {  
                if (mod1 <= n) res[i] = mod1, mod1 += 3;  
                else if (mod2 <= n) res[i] = mod2, mod2 += 3;  
                else res[i] = mod0, mod0 += 3;  
            }  
        }  
    }  
    else  
    {  
        for (int i = 1; i <= n; ++i)   
        {  
            if (col[i] == 1)  
            {  
                if (mod0 <= n) res[i] = mod0, mod0 += 3;  
            }  
            else  
            {  
                if (mod1 <= n) res[i] = mod1, mod1 += 3;  
                else if (mod2 <= n) res[i] = mod2, mod2 += 3;  
                else res[i] = mod0, mod0 += 3;  
            }  
        }  
    }  
    for (int i = 1; i <= n; ++i) printf("%d ", res[i]);  
    return 0;  
}  
```  
  
### CF1379F2 Chess Strikes Back (hard version)  
  
动态修改，看起来就是数据结构题。  
  
可能是线段树维护什么奇怪的东西。  
  
题目范围给定 $2n$，$2m$，都是偶数，可以尝试拆为 $2 \times 2$ 的小矩形来观察特性。  
  
限制对角放置，那么一个矩形有四种状态，即左上/右下是否可以放置。  
  
现在有一个 $n \times m$ 的大矩阵，对每一格都放置一个国王，只有两种选择：左上/右下。  
  
考虑什么时候会出现不合法的情况，定义只有左上可放置为 $L$，只有右下为 $R$ ，可以发现：  
  
一个 $L$ 会限制它同列上方所有点选择 $L$，一个 $R$ 会限制它同列下方所有点选择 $R$.  
  
一个 $L$ 会限制它同行左方所有点选择 $L$，一个 $R$ 会限制它同行右方所有点选择 $R$.  
  
一个 $L$ 会限制它对角线上左上方的所有点选择 $L$，一个 $R$ 会限制它对角线上右下方的所有点选择 $R$.  
  
有一个大胆的想法：对每行、每列、每个对角线开动态开点线段树，然后做单点修改、区间查询检验合法性。  
  
然而这样是不行的，因为一个点不仅有它自身的限制，还有它的限制造成的限制。  
  
例如左上角、右下角一个不在同对角线上的 $R$ 与 $L$，于是 $R$ 使它同行右侧的与 $L$ 同列的点强制选择 $R$，而与 $L$ 限制它同列上方的所有点选择 $L$ 冲突。  
  
考虑一个 $L$ 的所有限制都是对左上方的，而 $R$ 的所有限制都是对右下方的，可以发现冲突只可能发生在 $L$ 右下 $R$ 左上的情况中。  
  
这样就可以得到一个更紧的限制：  
  
当且仅当 $L$ 的左上方有 $R$ 时，不满足条件。  
  
这似乎就成了一个动态二维数点的问题？把时间序加进来，变成一个三维数点问题，硬上 cdq 分治就行了。  
  
时间复杂度 $O(n\log ^2 n)$.  
  
然而这样写太蠢了，因为只需要数有没有，不需要数个数。  
  
所以对于每一列，维护一个最小的 $R$ ，维护一个最大的 $L$ 就可以了。  
  
每次插入 $L$ 时，区间查询对应的 $R$，而每次插入 $R$ 时，区间查询对应的 $L$，再看看如何撤销操作。  
  
考虑暴力一点，对每一列，维护两个集合，然后暴力上就完事了。  
  
常数有点大的 $O(n \log n)$ 做法。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <set>  
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
const int maxn = 2e5 + 100;  
int n, m, q;  
int minr[maxn << 2], maxl[maxn << 2];  
bool cant[maxn << 2];  
set<int> LS[maxn], RS[maxn];  
#define ls p << 1  
#define rs p << 1 | 1  
void build(int l, int r, int p)  
{  
    minr[p] = 1 << 30, maxl[p] = 0;  
    if (l == r) return;  
    int mid = (l + r) >> 1;  
    build(l, mid, ls), build(mid + 1, r, rs);  
}  
inline void pushup(int p)  
{  
    cant[p] = cant[ls] | cant[rs] | (minr[ls] <= maxl[rs]);  
    minr[p] = min(minr[ls], minr[rs]);  
    maxl[p] = max(maxl[ls], maxl[rs]);  
}  
void update(int l, int r, int p, int pos)  
{  
    if(l == r)  
    {  
        minr[p] = RS[l].size() ? *RS[l].begin() : 1 << 30;  
        maxl[p] = LS[l].size() ? *--LS[l].end() : 0;  
        cant[p] = minr[p] <= maxl[p];  
        return;  
    }  
    int mid = (l + r) >> 1;  
    if (pos <= mid) update(l, mid, ls, pos);  
    else update(mid + 1, r, rs, pos);  
    pushup(p);  
}  
int main()  
{  
    read(n), read(m), read(q);  
    build(1, n, 1);  
    for (int i = 1, x, y; i <= q; ++i)  
    {  
        read(x), read(y);  
        int gx = (x + 1) >> 1, gy = (y + 1) >> 1;  
        if (x & 1)  
        {  
            //R  
            set<int>::iterator it = RS[gx].find(gy);  
            if (it == RS[gx].end()) RS[gx].insert(gy);  
            else RS[gx].erase(it);  
        }  
        else  
        {  
            //L  
            set<int>::iterator it = LS[gx].find(gy);  
            if (it == LS[gx].end()) LS[gx].insert(gy);  
            else LS[gx].erase(it);  
        }  
        update(1, n, 1, gx);  
        puts(cant[1] ? "NO" : "YES");  
    }  
    return 0;  
}  
```  
  
### CF1355C Count Triangles  
  
两小边之和大于第三边即可。  
  
$x + y > z$  
  
这就是本题唯一限制。  
  
一个很 naive 的想法就是枚举 $x$ 与 $y$，然后去计算有多少 $z$ 满足条件。  
  
然后根据数据范围猜测枚举 $x$，利用某些规律统计 $y$，才能在足够时间内通过。  
  
$$ \sum \limits _{y=B}^C (\max(\min (x+y-1,D) - C + 1),0) $$  
  
这个最小值函数显然是可以分段套出来的。  
  
$x + y - 1 \le D$ 时，即 $B \le y \le D - x + 1$ 时，贡献是 $\sum \limits _{y=B}^{\min(C,D-x+1)} (x + y - 1) - (D - x + 1 - B + 1) \times (C-1)$  
  
左边这团显然可以求和公式优化到 $O(1)$，那么这段统计可以 $O(1)$，然后再处理一下最大值，改一改端点即可。  
  
而当 $x + y - 1 > D$ 时，就更好做了：$C \ge y \ge D - x + 2$，然后贡献就是 $(C - (D - x + 2) + 1) \times (D - C + 1)$  
  
当然要判断一下空集的情况，总之随便搞搞就行了。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <ctype.h>  
const int bufSize = 1e6;  
#define DEBUG  
using std::min;  
using std::max;  
#define int long long  
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
const int maxn = 5e5 + 100;  
int A, B, C, D;  
signed main()  
{  
    read(A), read(B), read(C), read(D);  
    long long res = 0;  
    for (int x = A; x <= B; ++x)  
    {  
        int mid = min(D - x + 1, C);  
        if (mid >= B)  
        {  
            int num = mid - B + 1;  
            int left = x + B - 1, right = x + mid - 1;  
            left = max(left, C);  
            if (right >= left) num = right - left + 1, res += (left + right) * num / 2 - num * (C - 1);  
        }  
        mid = max(B,D - x + 2);  
        if(mid <= C)  
        {  
            int num = C - mid + 1;  
            res += num * (D - C + 1);  
        }  
    }  
    printf("%lld\n", res);  
    return 0;  
}  
```  
  
### CF1338B Edge Weight Assignment  
  
最少最多分开做。  
  
考虑怎么最少，有一个自然的想法是全部赋值为同一个数，如果叶子结点两两距离全都是偶数，这个就是对的。  
如果是奇数，考虑怎么做。可以发现至少需要三种。  
感受一下，应该也只需要三种就可以了。  
  
如何知道叶子结点两两距离是否偶数？记录子树内是否存在深度为奇/偶的叶子结点，然后在祖先处处理判定。  
  
最多可以有多少个呢？  
无论怎么异或，一个叶子都可以在它前往父亲的路上消掉所有的值，因此可以有很多。  
然而，同一个父亲有多个叶子的话，这些叶子往父亲的路上的值都是一样的。  
因此直接总边数减去 $\sum (leavenum(u) - 1)$ 即可。  
  
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
const int maxn = 1e5 + 100;  
int n, root, can, sum;  
struct node  
{  
    int to, next;  
} E[maxn << 1];  
int head[maxn];  
inline void add(const int& x, const int& y)  
{  
    static int tot = 0;  
    E[++tot].next = head[x],E[tot].to = y,head[x] = tot;  
};  
bool dep[maxn], f[maxn][2];  
void dfs1(int u, int fa)  
{  
    if (!E[head[u]].next) return (void)(f[u][dep[u]] = 1);  
    int sonleaf = 0;  
    for (int p = head[u]; p; p = E[p].next)  
    {  
        int v = E[p].to;  
        if(v == fa) continue;  
        dep[v] = dep[u] ^ 1, dfs1(v, u),sonleaf += !E[head[v]].next;  
        if ((f[v][0] && f[u][1]) || (f[v][1] && f[u][0])) can = 0;  
        f[u][0] |= f[v][0],f[u][1] |= f[v][1];  
    }  
    if (sonleaf) sum += sonleaf - 1;  
}  
int main()  
{  
    read(n);  
    if (n == 2) return puts("1 1"), 0;  
    for (int i = 1, a, b; i < n; ++i) read(a), read(b), add(a, b), add(b, a);  
    for (int i = 1; !root && i <= n; ++i) if (E[head[i]].next) root = i;  
    can = 1, dfs1(root, 0);  
    printf("%d %d\n", can ? 1 : 3, n - 1 - sum);  
    return 0;  
}  
```  
  
### CF1322B Present  
  
值域很奇怪，考虑按位处理。  
  
枚举第 $i$ 个数的第 $k$ 位，统计所有 $a_i + a_j$ 中第 $k$ 位为 $1$ 的个数，根据奇偶性判断答案第 $k$ 位的值。  
  
可以发现，第 $k$ 位的取值仅与前 $k$ 位有关。  
  
令 $b_i = a_i \text{ and } (2^{k+1} - 1)$，那么仅考虑 $b_j + b_i$ 即可确定第 $k$ 位取值。  
  
$b_i + b_j$ 何时第 $k$ 位为 $1$？当 $b_j + b_i \in [2^k,2^{k+1}-1]$ 时肯定成立，但还要考虑第 $k$ 位进位后又被第 $k-1$ 位进位为 $1$，因此 $b_j + b_i \in [2^{k+1} + 2^k,2^{k+2} - 2]$ 同样是合法的。  
  
这个东西当然可以大力树状数组维护，时间复杂度 $O(n \log^2 a_i)$，不太可行。  
  
可以考虑按 $b_i$ 排序，然后用双指针来统计个数，可以做到 $O(n \log n \log a_i)$，应该能过。  
  
然后这里排序可以用归并排序，这样就能稳定通过了。  
  
时间复杂度 $O(n \log a_i)$.  
  
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
template <typename T>  
inline T read(T& r)  
{  
    static char c;  
    r = 0;  
    for (c = nc(); !isdigit(c); c = nc());  
    for (; isdigit(c); c = nc()) r = r * 10 + c - 48;  
    return r;  
}  
const int maxn = 4e5 + 100;  
int n, a[maxn], b[maxn], id[maxn], maxx, ans;  
int L[maxn], Lid[maxn], R[maxn], Rid[maxn];  
void mergesort(int k)  
{  
    int lnum = 0, rnum = 0;  
    for (int i = 1; i <= n; ++i)  
        if (a[id[i]] & (1 << k)) R[++rnum] = b[i] | (1 << k), Rid[rnum] = id[i];  
        else L[++lnum] = b[i], Lid[lnum] = id[i];  
    int p1 = 1, p2 = 1, p = 1;  
    while (p1 <= lnum && p2 <= rnum)  
        if (L[p1] <= R[p2]) b[p] = L[p1], id[p++] = Lid[p1++];  
        else b[p] = R[p2], id[p++] = Rid[p2++];  
    while (p1 <= lnum) b[p] = L[p1], id[p++] = Lid[p1++];  
    while (p2 <= rnum) b[p] = R[p2], id[p++] = Rid[p2++];  
}  
int count(int ll, int rr)  // count how many in [ll,rr] using two pointer  
{  
    int bitnum = 0, l = 1, r = n;  
    for (int i = n; i; L[i--] = l)  
        while (l < i && b[l] + b[i] < ll) ++l;  
    for (int i = 1; i <= n; R[i] = std::min(i - 1, r), ++i)  
        while (r > 0 && b[i] + b[r] > rr) --r;  
    for (int i = 2; i <= n; ++i)  
        if (R[i] >= L[i]) bitnum += R[i] - L[i] + 1;  
    return bitnum;  
}  
int main()  
{  
    read(n);  
    for (int i = 1; i <= n; ++i) read(a[i]), maxx = std::max(maxx, a[i]);  
    for (int i = 1; i <= n; ++i) id[i] = i;  
    for (int k = 0; (1 << k) <= (maxx << 1); ++k)  
    {  
        mergesort(k);  
        int bitnum = count(1 << k, (1 << (k + 1)) - 1) + count((1 << k) + (1 << (k + 1)), (1 << (k + 2)) - 2);  
        if (bitnum & 1) ans |= 1 << k;  
    }  
    printf("%d\n", ans);  
    return 0;  
}  
```  
  
### CF1304C Air Conditioner  
  
每个时刻可能的温度区间是固定的，可以直接模拟一遍过去。  
  
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
const int maxn = 110;  
int T, n, m, L[maxn], R[maxn], pos[maxn];  
int main()  
{  
    read(T);  
    while(T--)  
    {  
        read(n),read(m);  
        for (int i = 1; i <= n; ++i) read(pos[i]), read(L[i]), read(R[i]);  
        for (int i = n; i; --i) pos[i] -= pos[i - 1];  
        int l = m, r = m;  
        for (int i = 1; i <= n; ++i)  
        {  
            l = std::max(l - pos[i], L[i]), r = std::min(r + pos[i], R[i]);  
            if (l > r) goto fail;  
        }  
        puts("YES");  
        continue;  
        fail:  
        puts("NO");  
    }  
    return 0;  
}  
```  
  
### CF1292B Aroma's Search  
  
可以发现，每次坐标至少翻倍，因此暴力复杂度是对数级别的。  
  
那么暴力把所有点模拟出来。  
  
然后大力搜索，大力剪枝。  
  
然而这样还是会挂，因为点太多了，暴力搜索是 $O(n!)$ 的。  
  
大力猜结论，每次只会移动到相邻的点，因为走曼哈顿距离，非相邻的点一定可以从相邻的点走过去，且距离不变。  
  
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
#define ll long long  
ll x0, y0, ax, ay, bx, by, sx, sy, t;  
const ll maxx = 1e18;  
const int maxn = 200;  
ll X[maxn], Y[maxn], cnt, res;  
ll d[maxn][maxn];  
ll myabs(ll x) { return x > 0 ? x : -x; }  
ll getdis(ll x1, ll y1, ll x2, ll y2) { return myabs(x1 - x2) + myabs(y1 - y2); }  
bool vis[200];  
void dfs(int pos, ll num, ll nowt)  
{  
    res = std::max(res, num);  
    for (int i = 1; i <= cnt; ++i)  
        if (myabs(pos - i) < 2 && !vis[i] && nowt >= d[i][pos])  
        {  
            vis[i] = 1;  
            dfs(i, num + 1, nowt - d[i][pos]);  
            vis[i] = 0;  
        }  
}  
int main()  
{  
    read(x0), read(y0), read(ax), read(ay), read(bx), read(by), read(sx), read(sy), read(t);  
    ll nowx = x0, nowy = y0;  
    for (int i = 0;; ++i)  
    {  
        if (getdis(nowx, nowy, sx, sy) <= t) X[++cnt] = nowx, Y[cnt] = nowy;  
        if ((maxx - bx) / nowx < ax) break;  
        if ((maxx - by) / nowy < ay) break;  
        nowx = nowx * ax + bx, nowy = nowy * ay + by;  
    }  
    for (int i = 1; i <= cnt; ++i)  
        for (int j = i + 1; j <= cnt; ++j) d[i][j] = d[j][i] = getdis(X[i], Y[i], X[j], Y[j]);  
    //printf("%lld\n",cnt);  
    for (int i = 1; i <= cnt; ++i)  
        if (t >= getdis(sx, sy, X[i], Y[i]))  
        {  
            vis[i] = 1;  
            dfs(i, 1, t - getdis(sx, sy, X[i], Y[i]));  
            vis[i] = 0;  
        }  
    printf("%lld\n", res);  
    return 0;  
}  
```  
  
### CF1320C World of Darkraft: Battle for Azathoth  
  
一个很自然的想法是枚举一个端点，二分另一个端点，不幸的是没有什么单调性可言。  
  
考虑依然枚举一个端点，然后上数据结构。  
  
定义 $f(i)$ 为在当前枚举的 $x$ 下，选择 $y = i$ 时能获得的最大价值。每次把新的 $x$ 的怪物的收益插入，把对应的新的 $x$ 的武器的代价插入即可。  
  
初始化时，把代价初始化进去。  
  
维护一个线段树，区间修改，全局最大。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <vector>  
#include <cstring>  
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
const int maxn = 2e5 + 100;  
const int maxm = 1e6;  
int n, m, p;  
int costa[maxm + 100], costb[maxm + 100];  
vector<pair<int,int> > V[maxm + 100];  
int maxx[maxm << 2], tag[maxm << 2];  
#define ls p << 1  
#define rs p << 1 | 1  
inline void pushup(int p) { maxx[p] = max(maxx[ls], maxx[rs]); }  
inline void pushdown(int p) { if (tag[p]) maxx[ls] += tag[p], maxx[rs] += tag[p], tag[ls] += tag[p], tag[rs] += tag[p], tag[p] = 0; }  
void build(int l, int r, int p)  
{  
    if (l == r) return (void)(maxx[p] = -costb[l]);  
    int mid = l + r >> 1;  
    build(l, mid, ls), build(mid + 1, r, rs), pushup(p);  
}  
void modify(int l,int r,int p,int ll,int rr,int k)  
{  
    if (l >= ll && r <= rr) return (void)(maxx[p] += k, tag[p] += k);  
    pushdown(p);  
    int mid = (l + r) >> 1;  
    if (ll <= mid) modify(l, mid, ls, ll, rr, k);  
    if (rr > mid) modify(mid + 1, r, rs, ll, rr, k);  
    pushup(p);  
}  
int main()  
{  
    read(n), read(m), read(p);  
    memset(costa, 0x3f, sizeof(costa)), memset(costb, 0x3f, sizeof(costb));  
    int maxa = 0, maxb = 0;  
    for (int i = 1, a, cost; i <= n; ++i) read(a), read(cost), costa[a] = min(costa[a], cost), maxa = max(maxa, a);  
    for (int i = 1, b, cost; i <= m; ++i) read(b), read(cost), costb[b] = min(costb[b], cost), maxb = max(maxb, b);  
    for (int i = maxa - 1; i; --i) costa[i] = min(costa[i], costa[i + 1]);  
    for (int i = maxb - 1; i; --i) costb[i] = min(costb[i], costb[i + 1]);  
    for (int i = 1, x, y, z; i <= p; ++i)  
        read(x), read(y), read(z), V[x].push_back(make_pair(y, z));  
    build(1, maxb, 1);  
    int res = -2100000000;  
    for (int i = 1; i <= maxa; ++i)  
    {  
        res = max(res, maxx[1] - costa[i]);  
        for (vector<pair<int, int> >::iterator it = V[i].begin(); it != V[i].end(); ++it)  
            if (it->first < maxb) modify(1, maxb, 1, it->first + 1, maxb, it->second);  
    }  
    printf("%d\n", res);  
    return 0;  
}  
```  
  
### CF1304E 1-Trees and Queries  
  
将路径分为两类：经过额外边与不经过额外边。  
  
无向边成二环，只需要考虑奇偶性。  
  
经过额外边也有两种可能，即从两个端点进入，随便搞搞即可。这破题怎么都到 E 来了。  
  
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
const int maxn = 1e5 + 100;  
int n, m;  
struct node  
{  
    int to, next;  
} E[maxn << 1];  
int head[maxn];  
inline void add(const int& x, const int& y)  
{  
    static int tot = 0;  
    E[++tot].next = head[x], E[tot].to = y, head[x] = tot;  
}  
int size[maxn], son[maxn], fa[maxn], dep[maxn], top[maxn];  
void dfs1(int u)  
{  
    size[u] = 1;  
    for (int p = head[u]; p; p = E[p].next)  
    {  
        int v = E[p].to;  
        if (v == fa[u]) continue;  
        dep[v] = dep[u] + 1, fa[v] = u, dfs1(v), size[u] += size[v];  
        if (size[v] > size[son[u]]) son[u] = v;  
    }  
}  
void dfs2(int u)  
{  
    if (!son[u]) return;  
    top[son[u]] = top[u], dfs2(son[u]);  
    for (int p = head[u]; p; p = E[p].next)  
    {  
        int v = E[p].to;  
        if (v == fa[u] || v == son[u]) continue;  
        dfs2(top[v] = v);  
    }  
}  
inline int lca(int x, int y)  
{  
    for (; top[x] != top[y]; x = fa[top[x]]) if (dep[top[x]] < dep[top[y]]) std::swap(x, y);  
    return dep[x] < dep[y] ? x : y;  
}  
inline int getdis(int x, int y) { return dep[x] + dep[y] - (dep[lca(x, y)] << 1); }  
inline bool check(int x, int y) { return x <= y && (x & 1) == (y & 1); }  
int main()  
{  
    read(n);  
    for (int i = 1, a, b; i < n; ++i) read(a), read(b), add(a, b), add(b, a);  
    dfs1(1), top[1] = 1, dfs2(1);  
    read(m);  
    for (int i = 1, x, y, a, b, k; i <= m; ++i)  
    {  
        read(x), read(y), read(a), read(b), read(k);  
        //not passing  
        int d = getdis(a, b);  
        if (check(d,k)) { puts("YES"); continue; }  
        //goto x  
        d = getdis(a, x) + 1 + getdis(y, b);  
        if (check(d, k)) { puts("YES"); continue; }  
        //goto y  
        d = getdis(a, y) + 1 + getdis(x, b);  
        if (check(d, k)) { puts("YES"); continue; }  
        puts("NO");  
    }  
    return 0;  
}  
```  
  
## 结语  
  
35 题的题单肝了 29 题，有点难受，各种猜结论、构造做的有点晕了，所以就告一段落了。下一阶段以刷真题为主。