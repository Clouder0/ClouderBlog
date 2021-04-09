---  
title: '[Atcoder][CF]简单题选做练习笔记'    
date: 2020-11-12T14:33:35+08:00  
slug: "atcftrain"  
type: post  
tags:  
  - atcoder  
  - codeforces  
categories:  
  - Algorithm  
---  
  
  
## 前言  
  
板刷题单：[https://www.luogu.com.cn/training/2018#problems](https://www.luogu.com.cn/training/2018#problems)  
  
### AT5741 Fusing Slimes  
  
可以想到，每种情况都对应一个长度为 $n - 1$ 的排列。  
  
考虑计算每条路径被经过了多少次， 先对原数组做差分。  
  
记 $d(i)$ 为 $i \to i + 1$ 的长度，$f(i)$ 为 $i \to i + 1$ 的路径被经过的期望次数。  
  
$E = \sum \limits _{i = 1}^{n-1}d(i) \times f(i)$  
  
考虑如何计算 $f(i)$。  
  
定义 $g(j,i)$ 为点 $j$ 向右融合经过 $i \to i + 1$ 这条路径的期望次数。  
  
有：$f(i) = \sum \limits _{j = 1}^i g(j,i)$  
  
问题在于计算 $g(j,i)$，考虑 $j$ 点能顺畅地右移到 $i + 1$，那么 $[j + 1,i]$ 间的点都已经右移。  
  
表现在序列上，就是 $j$ 的出现位置一定在 $[j + 1,i]$ 之后。  
  
可以考虑先在移动序列中对 $[j,i]$ 间的 $i - j + 1$ 个元素去序，然后将 $j$ 放在最后，剩余 $i - j$ 个元素进行全排列，放回原序列中。  
  
$$ g(j,i) = \dfrac{(n-1)!}{(i - j + 1)!} \times (i-j)!  
$$  
  
$$ g(j,i) = \dfrac{(n-1)!}{i-j+1}  
$$  
  
这样就较为简单地推出了 $g(j,i)$，而不需要别的题解中我看不懂的奇怪式子。(  
  
$$ f(i) = \sum \limits _{j=1}^i g(j,i) = (n-1)! \times \sum \limits _{j=1}^i \dfrac{1}{i-j+1}  
$$  
  
还可以进一步化简：  
  
$$ f(i) = (n-1)! \times \sum \limits _{j=1}^i \dfrac{1}{j}  
$$  
  
而答案就是：  
  
$$  
E = \sum d(i) \times f(i) = (n - 1)! \times \sum \limits _{i=1}^{n-1} (d(i) \times \sum \limits _{j=1}^i \dfrac{1}{j})  
$$  
  
可以线性递推逆元，预处理调和级数前缀和做到 $O(n)$ 复杂度。  
  
```cpp  
const int maxn = 1e5 + 100;  
const int mod = 1e9 + 7;  
inline int add(int x, int y) { int res = x + y; return res >= mod ? res - mod : res; }  
inline int mul(int x, int y) { return 1ll * x * y % mod; }  
int n, a[maxn], inv[maxn], sum[maxn], res, prod = 1;  
int main()  
{  
    read(n);  
    for (int i = 1; i <= n; ++i) read(a[i]);  
    inv[1] = sum[1] = 1;  
    for (int i = 2; i <= n; ++i) sum[i] = add(sum[i - 1], inv[i] = mul(inv[mod % i], mod - mod / i));  
    for (int i = 1; i < n; ++i) prod = mul(prod, i), res = add(res, mul(a[i + 1] - a[i], sum[i]));  
    printf("%d\n", mul(prod, res));  
    return 0;  
}  
```  
  
### AT4815 [ABC154E] Almost Everywhere Zero  
  
$K \le 3$，怎么浓浓的一股分类讨论气味……  
  
虽然一眼觉得是数位 DP……  
  
然而我觉得数位 DP 是可以的，于是打之。  
  
确实是可以的。那这题有点水。  
  
```cpp  
#include <cstdio>  
#include <cstring>  
const int maxn = 110;  
char s[maxn];  
int num[maxn], n, k;  
long long f[maxn][4][2][2];  
long long dfs(int pos, int limit, int zero, int sum)  
{  
    if (sum > k) return 0;  
    if (f[pos][sum][limit][zero] != -1) return f[pos][sum][limit][zero];  
    if (pos == n + 1) return sum == k;  
    int up = limit ? num[pos] : 9;  
    long long res = 0;  
    for (int i = 0; i <= up; ++i) res += dfs(pos + 1, i == up && limit, zero && !i, sum + (i > 0));  
    return f[pos][sum][limit][zero] = res;  
}  
int main()  
{  
    scanf("%s\n%d", s + 1, &k);  
    memset(f, -1, sizeof(f));  
    n = std::strlen(s + 1);  
    for (int i = 1; i <= n; ++i) num[i] = s[i] - '0';  
    printf("%lld\n", dfs(0, 1, 1, 0));  
    return 0;  
}  
```  
  
### AT5295 [ABC154F] Many Many Paths  
  
单个 $f(i,j)$ 可以直接通过组合数计算。考虑在 $i+j$ 步操作中选出 $i$ 步向下即可。  
  
$$ f(i,j) = \dbinom{i+j}{i}  
$$  
  
然后考虑如何求解下面这个式子：  
  
$$ \sum \limits _{i=r_1}^{r_2} {\sum \limits _{j=c_1}^{c_2} f(i,j)}  
$$  
  
$$ \sum \limits _{i=r_1}^{r_2} {\sum \limits _{j=c_1}^{c_2} \dbinom{i+j}{i}}  
$$  
  
只有上半部分变化，看起来是有一定性质的。  
  
考虑先把 $j$ 改成从零开始，这样比较有意义，所以要做一个前缀和一样的减法。  
  
$$ \sum \limits _{i=r_1}^{r_2} ({\sum \limits _{j=0}^{c_2} \dbinom{i+j}{i}} - {\sum \limits _{j=0}^{c_1 - 1} \dbinom{i+j}{i}})  
$$  
  
问题转化为，计算：  
  
$$ \sum \limits _{j=0}^k \dbinom{i+j}{i}  
$$  
  
这个其实就是 *Hockey-Stick Identity* 反过来。  
  
$$\dbinom{i + k + 1}{i + 1} = \sum \limits _{j=0}^k \dbinom{i+j}{i}  
$$  
  
证明的话，可以考虑将待选择序列划分为 $[1,i]$ 与 $[i + 1,i + k + 1]$，然后枚举从右往左第一个选择的物品。  
  
这样就可以大幅简化计算了。  
  
$$\sum \limits _{i=r_1}^{r_2} \dbinom{i+c_2+1}{i+1} - \dbinom{i+c_1}{i+1}  
$$  
  
还可以把组合数拆开。  
  
$$\sum \limits _{i=r_1}^{r_2} \dfrac{(i+c_2+1)!}{c_2!(i+1)!} - \dfrac{(i+c_1)!}{(c_1-1)!(i+1)!}  
$$  
  
接下来大家都知道怎么做了吧。  
  
```cpp  
const int mod = 1e9 + 7;  
const int maxn = 2e6 + 100;  
int r1, r2, c1, c2, prod[maxn], prodinv[maxn], inv[maxn], res;  
inline int add(int x, int y) { int res = x + y; return res >= mod ? res - mod : res; }  
inline int mul(int x, int y) { return 1ll * x * y % mod; }  
inline int c(int n, int m) { return mul(prod[n], mul(prodinv[n - m], prodinv[m])); }  
int main()  
{  
    read(r1), read(c1), read(r2), read(c2);  
    inv[1] = 1;  
    int up = r2 + c2 + 10;  
    prod[0] = prodinv[0] = prod[1] = prodinv[1] = 1;  
    for (int i = 2; i <= up; ++i)  
        prod[i] = mul(prod[i - 1], i), prodinv[i] = mul(prodinv[i - 1], inv[i] = mul(inv[mod % i], mod - mod / i));  
    for (int i = r1; i <= r2; ++i)  
        res = add(res, add(c(i + c2 + 1, i + 1), mod - c(i + c1, i + 1)));  
    printf("%d\n", res);  
    return 0;  
}  
```  
  
### AT4867 [ABC155D] Pairs  
  
神秘题。  
没啥想法，以为是结论题。看了题解才知道是二分。  
知道是二分就好做了。  
  
讨论一下，然后根据单调性双指针， 分为 负负，正负，正正分别讨论。  
  
```cpp  
#include <algorithm>  
#include <cstdio>  
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
template <typename T>  
inline T read(T& r)  
{  
    static char c;  
    static int flag;  
    flag = 1, r = 0;  
    for (c = nc(); !isdigit(c); c = nc()) if (c == '-') flag = -1;  
    for (; isdigit(c); c = nc()) r = r * 10 + c - 48;  
    return r *= flag;  
}  
const int maxn = 2e5 + 100;  
int n, k, a[maxn], zeromid;  
int check(long long mid)  
{  
    int p = zeromid, res = 0;  
    for (int i = 1; i < zeromid; ++i)  
    {  
        while (1ll * a[i] * a[p] > mid && p <= n) ++p;  
        if (p <= n) res += (n - p + 1);  
    }  
    p = n;  
    for (int i = zeromid; i <= n; ++i)  
    {  
        while (1ll * a[i] * a[p] > mid && p > 0) --p;  
        if (p > i) res += p - i;  
    }  
    p = 1;  
    for (int i = zeromid - 1; i > 0; --i)  
    {  
        while (1ll * a[i] * a[p] > mid && p < zeromid) ++p;  
        if (p < i) res += i - p;  
    }  
    return res;  
}  
template <typename T>  
inline T abs(const T& x)  
{  
    return x > 0 ? x : -x;  
}  
signed main()  
{  
    read(n), read(k);  
    for (int i = 1; i <= n; ++i) read(a[i]);  
    std::sort(a + 1, a + 1 + n);  
    int maxx = std::max(abs(a[n]), abs(a[1]));  
    long long l = -(maxx * maxx), r = maxx * maxx, mid, ans = 0;  
    for (int i = 1; !zeromid && i <= n; ++i) if (a[i] >= 0) zeromid = i;  
    while (l <= r)  
    {  
        mid = (l + r) >> 1;  
        if (check(mid) >= k) ans = mid, r = mid - 1;  
        else l = mid + 1;  
    }  
    printf("%lld\n", ans);  
    return 0;  
}  
```  
  
### AT4866 [ABC155E] Payment  
  
看上去就是动态规划之类的解法吧。  
题意转化为 $x \ge 0$，最小化 $f(x + N) + f(x)$，然后考虑。  
对于 $f(x + N)$，从后往前，每位对下一位的影响就是是否进位，而且进位也只进一位。  
  
而 $x$ 的这位填多少？考虑填完之后，要么 $x + N$ 这一位为零，要么 $x$ 这一位为零。  
塞到状态里就可以了。  
  
$g(i,0)$ 代表第 $i$ 位未进位， $g(i,1)$ 代表第 $i$ 位向 $i+1$ 进位了，然后转移随便搞搞就行了。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <cstring>  
#include <ctype.h>  
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
    for (; !isdigit(c); c = nc());  
    for (; isdigit(c); c = nc()) *s++ = c;  
    *s = '\0';  
}  
const int maxn = 1e6 + 100;  
char s[maxn];  
int n, f[maxn][2];  
int main()  
{  
    read(s + 1);  
    n = strlen(s + 1);  
    for (int i = 1; i <= n; ++i) s[i] -= '0';  
    f[n][0] = s[n],f[n][1] = 10 - s[n];  
    for(int i = n - 1;i > 0;--i)  
    {  
        f[i][1] = min(f[i + 1][0] + 10 - s[i], f[i + 1][1] + 9 - s[i]);  
        if (s[i] != '9') f[i][0] = min(f[i + 1][0] + s[i], f[i + 1][1] + s[i] + 1);  
        else f[i][0] = f[i + 1][0] + s[i];  
    }  
    printf("%d\n",min(f[1][0],f[1][1] + 1));  
    return 0;  
}  
```  
  
### AT4831 [ABC155F] Perils in Parallel  
  
看上去就很神的题，感觉很老，但就是不会做。  
  
首先，区间异或，可以发现异或满足差分性质，因此对 $[l,r]$ 区间异或可以拆成在差分数组上对 $l$ 与对 $r + 1$ 异或。  
  
感觉很图论，令图上每个点都代表 $i$ 点，然后对 $l$ 点与 $r + 1$ 点连边，代表可以通过该边翻转颜色。  
  
如果选择了该操作，就选择了该边。  
  
一个点的最终值，与它相连的边的奇偶性有关。  
  
对于每一个联通块，相连的两个黑点总是可以同时变白，而中间的点总是可以不改变状态。那么统计黑点数量，如果可以两两消去就存在解。  
  
那么如何统计方案呢？可以用类似差分的方法标记路径，每次遇到一个黑点就打标记往上消，然后遇到另一个黑点的标记，消了两次就相当于没变化了。  
  
答案还要按从小到大输出。  
  
```cpp  
#include <algorithm>  
#include <cstdio>  
#include <ctype.h>  
using namespace std;  
#define int long long  
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
const int maxn = 4e5 + 100;  
const int maxm = 4e5 + 100;  
int n, m, cnt, d[maxn], p[maxn], s[maxn], fa[maxn], onenum, toid[maxn], vis[maxn];  
struct light  
{  
    int pos, status;  
} A[maxn];  
inline bool cmp(const light& a, const light& b) { return a.pos < b.pos; }  
struct node  
{  
    int to, next, val;  
} E[maxm];  
int head[maxn];  
inline void add(const int& x, const int& y, const int& val)  
{  
    static int tot = 0;  
    E[++tot].next = head[x], E[tot].to = y, E[tot].val = val, head[x] = tot;  
}  
void dfs(int u)  
{  
    if (s[u]) d[u] = 1, ++onenum;  
    for (int p = head[u], v; p; p = E[p].next)  
        if (!fa[v = E[p].to]) fa[v] = u, toid[v] = E[p].val, dfs(v), d[u] += d[v];  
}  
signed main()  
{  
    read(n),read(m);  
    for (int i = 1; i <= n; ++i) read(A[i].pos), read(A[i].status);  
    sort(A + 1, A + 1 + n,cmp);  
    for (int i = 1; i <= n; ++i) p[i] = A[i].pos, s[i] = A[i].status;  
    for (int i = n + 1; i; --i) s[i] = s[i] ^ s[i - 1];  
    for (int i = 1, l, r; i <= m; ++i)  
    {  
        read(l), read(r), l = lower_bound(p + 1, p + 1 + n, l) - p, r = upper_bound(p + 1, p + 1 + n, r) - p - 1;  
        if (l <= r) add(l, r + 1, i), add(r + 1, l, i);  
    }  
    for (int i = 1; i <= n + 1; ++i)  
        if (!fa[i])  
        {  
            onenum = 0, fa[i] = i, dfs(i);  
            if (onenum & 1) goto fail;  
        }  
    for (int i = 1; i <= n + 1; ++i) if (d[i] & 1) vis[toid[i]] = 1,++cnt;  
    printf("%lld\n", cnt);  
    for (int i = 1; i <= m; ++i) if(vis[i]) printf("%lld ",i);  
    return 0;  
fail:  
    puts("-1");  
    return 0;  
}  
```  
  
### AT5312 [ABC156E] Roaming  
  
移动恰好 $k$ 步很有意思，然后发现每个人移动的唯一限制就是不能是上一个。  
  
然后可以发现两步成环，三步也能成环，那么 $k \ge 2$ 时可以任意移动。然后就不知道有什么限制了。  
  
看看题解怎么说。  
  
设最终态人数为 $v_i$，那么 $v_i = 0$ 的个数不得超过 $k$ 个，因为每步最多创造一个 $0$ 位置。  
  
然后就没有别的限制了。。。  
  
最多 $k$ 个零，解 $\sum x_i = n$ 的个数，考虑枚举零的个数，强制选零后再计算。  
  
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
const int mod = 1e9 + 7;  
const int maxn = 2e5 + 100;  
inline int add(int x, int y) { int res = x + y; return res >= mod ? res - mod : res; }  
inline int mul(int x, int y) { return 1ll * x * y % mod; }  
int n, k, prod[maxn], inv[maxn], prodinv[maxn], res;  
inline int C(int n, int m) { return mul(prod[n], mul(prodinv[n - m], prodinv[m])); }  
int main()  
{  
    read(n), read(k);  
    if(k >= n) k = n - 1;  
    prod[0] = prodinv[0] = prod[1] = prodinv[1] = inv[1] = 1;  
    for (int i = 2; i <= n; ++i) prod[i] = mul(prod[i - 1], i), prodinv[i] = mul(prodinv[i - 1], inv[i] = mul(inv[mod % i], mod - mod / i));  
    for (int i = 0; i <= k; ++i) res = add(res, mul(C(n, i), C(n - 1, n - i - 1)));  
    printf("%d\n", res);  
    return 0;  
}  
```  
  
### AT5334 [ABC156F] Modularness  
  
首先有 $d_i \ge 0$，开始考虑。  
  
如果 $d_i = 0$，那么对应的 $j$ 一定不满足。如果 $d_i > 0$，那么对应的 $j$ 有可能满足，具体要看取模。  
  
如果不考虑取模，那么对应的 $j$ 一定有 $a_j < a_{j + 1}$，而考虑取模后有可能不成立。可以发现，仅当进位后，有可能不成立，但也只是有可能。因为可能原本值很大，加上后取模也大，那么列个式子：$a_j < a_j + d_i - m$，就可以发现 $d_i > m$ 是不可能的。  
  
那么当且仅当 $a_{j} + d_i \ge m$ 时，有 $a_j > a_{j + 1}$，而当且仅当 $d_i = 0$ 时，有 $a_j = a_{j+1}$.  
  
只要能够统计这两种，就能减出我们需要的答案了。  
  
$d_i = 0$ 很好统计，算算有多少个对应的 $j$ 即可。  
  
而 $d_i > 0$，需要计算进位，可以直接用总和去除。  
  
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
const int maxn = 5100;  
int k, q, d[maxn], nd[maxn];  
long long sum[maxn];  
int main()  
{  
    read(k),read(q);  
    for (int i = 1; i <= k; ++i) read(d[i]);  
    for (int i = 1, n, x, m; i <= q; ++i)  
    {  
        read(n), read(x), read(m);  
        --n;  
        int res = n, zeronum = 0;  
        for (int j = 1; j <= k; ++j) nd[j] = d[j] % m, sum[j] = sum[j - 1] + nd[j], zeronum += !nd[j];  
        res -= n / k * zeronum;  
        for (int j = 1; j <= n % k; ++j) res -= !nd[j];  
        long long tsum = x + n / k * sum[k] + sum[n % k];  
        res -= tsum / m - x / m;  
        printf("%d\n",res);  
    }  
    return 0;  
}  
```  
  
### AT5369 [ABC158F] Removing Robots  
  
考虑正难则反，计算出有多少个激活集合。  
  
每一个激活集合会对应一个未激活集合。  
  
然后可以发现，当激活某一个机器人时，会激活它后方的一系列机器人，因此可以考虑把它们缩在一起。  
  
可以简单地用一个动态规划来实现：  
  
定义 $f(i,0)$ 为考虑 $i$ 以后的所有机器人，不激活 $i$ 时的方案数，$f(i,1)$ 为激活 $i$ 时的方案数。  
  
有：  
  
$$ f(i,0) = f(i + 1,0) + f(i + 1,1)  
$$  
  
而激活的话，假定激活 $i$ 后区间 $[i,to(i))$ 都会激活，那么从 $to(i)$ 转移即可。  
  
$$ f(i,1) = f(to(i),0) + f(to(i),1)  
$$  
  
只需要求出 $to(i)$ 就解决了，可以用二分求出 $i$ 点可以直接激活的边界，然后查询区间最值即可。  
  
随便打了个线段树。  
  
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
const int mod = 998244353;  
int n, to[maxn], f[maxn][2];  
struct node  
{  
    int pos, d;  
} A[maxn];  
bool cmp(const node& a, const node& b) { return a.pos < b.pos; }  
int maxx[maxn << 2];  
#define ls p << 1  
#define rs p << 1 | 1  
inline void pushup(int p) { maxx[p] = std::max(maxx[ls], maxx[rs]); }  
void modify(int l, int r, int p, int pos, int k)  
{  
    if(l == r) return (void)(maxx[p] = k);  
    int mid = l + r >> 1;  
    if (pos <= mid) modify(l, mid, ls, pos, k);  
    else modify(mid + 1, r, rs, pos, k);  
    pushup(p);  
}  
int ask(int l, int r, int p, int ll, int rr)  
{  
    if (ll > rr) return 0;  
    if (l >= ll && r <= rr) return maxx[p];  
    int mid = l + r >> 1, res = 0;  
    if (ll <= mid) res = ask(l, mid, ls, ll, rr);  
    if (rr > mid) res = std::max(res, ask(mid + 1, r, rs, ll, rr));  
    return res;  
}  
int main()  
{  
    read(n);  
    for (int i = 1; i <= n; ++i) read(A[i].pos), read(A[i].d);  
    std::sort(A + 1, A + 1 + n, cmp);  
    for (int i = n; i; --i)  
    {  
        int l = i, r = n, mid, ans = 0;  
        while (l <= r)  
        {  
            mid = l + r >> 1;  
            if (A[mid].pos < A[i].pos + A[i].d) ans = mid, l = mid + 1;  
            else r = mid - 1;  
        }  
        to[i] = std::max(r + 1, ask(1, n, 1, i + 1, ans));  
        modify(1, n, 1, i, to[i]);  
    }  
    f[n + 1][0] = 1;  
    for (int i = n; i; --i)  
    {  
        f[i][0] = (f[i + 1][0] + f[i + 1][1]) % mod;  
        f[i][1] = (f[to[i]][0] + f[to[i]][1]) % mod;  
    }  
    printf("%d\n", (f[1][0] + f[1][1]) % mod);  
    return 0;  
}  
```  
  
### AT5240 [ABC160E] Red and Green Apples  
  
同为 ABC 的 E 题，这题水的不行啊。  
直接按权值从大到小一起排序，然后逐个取即可。  
  
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
const int maxn = 1e6 + 100;  
int X, Y, A, B, C;  
int p[maxn], q[maxn], a[maxn],cnt;  
int main()  
{  
    read(X),read(Y),read(A),read(B),read(C);   
    for (int i = 1; i <= A; ++i) read(p[i]);  
    for (int i = 1; i <= B; ++i) read(q[i]);  
    for (int i = 1; i <= C; ++i) read(a[++cnt]);  
    std::sort(p + 1, p + 1 + A), std::sort(q + 1, q + 1 + B);  
    for (int i = 1; i <= X; ++i) a[++cnt] = p[A - i + 1];  
    for (int i = 1; i <= Y; ++i) a[++cnt] = q[B - i + 1];  
    std::sort(a + 1,a + 1 + cnt);  
    long long res = 0;  
    for (int i = 1; i <= X + Y; ++i) res += a[cnt - i + 1];  
    printf("%lld\n", res);  
    return 0;  
}  
```  
  
### AT4827 [ABC165D] Floor Function  
  
看上去很有意思。其实可以比较简单地推导出来。  
  
$$\lfloor\dfrac{ax}{b} \rfloor - a\times \lfloor\dfrac{x}{b} \rfloor  
$$  
  
$$ \implies \lfloor \dfrac{a \times (\lfloor \dfrac{x}{b} \rfloor \times b + x \bmod b)}{b} \rfloor - a \times \lfloor \dfrac{x}{b} \rfloor  
$$  
  
$$ \implies a \times \lfloor \dfrac{x}{b} \rfloor + \lfloor \dfrac{a \times (x \bmod b)}{b} \rfloor - a \times \lfloor \dfrac{x}{b}\rfloor  
$$  
  
$$ \implies \lfloor \dfrac{a \times (x \bmod b)}{b} \rfloor  
$$  
  
最大化这东西，取 $b - 1$ 即可。如果取不到，则尽量大。  
  
```cpp  
printf("%lld\n", 1ll * a * std::min(n, b - 1) / b);  
```  
  
### AT5239 [ABC166F] Three Variables Game  
  
一眼看上去很图论，然而只有三个变量，而且要求也不太靠谱，所以可以随便 YY 一下。  
要求是对每次操作做出决策，有一个显然的贪心是每次选较大的减，较小的加。  
不幸的是，这样会挂，而且挂的测试点名称都叫做 killer....  
  
这个做法有一个漏洞，当两个数大小相同时，随便选一个的话，贪心性质就没了。  
  
那么让看起来对后面贡献最大的增大，另一个没那么有用的减小可以吗？  
根据后一个操作判断贡献即可，如果后一个操作包含当前某点，那么就增大那个点，用来给下面的分配。如果后面操作与当前相同，那么随便改，后面可以反悔改回来。  
  
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
const int maxn = 1e5 + 100;  
int n, A, B, C, opt[maxn], sum[4][maxn];  
int AB, AC, BC;  
char ans[maxn];  
int main()  
{  
    read(n), read(A), read(B), read(C);  
    for (int i = 1; i <= n; ++i)  
    {  
        static char s[10];  
        read(s);  
        if (s[0] == 'A' && s[1] == 'B') opt[i] = 1;  
        else if (s[0] == 'B' && s[1] == 'C') opt[i] = 2;  
        else if (s[0] == 'A' && s[1] == 'C') opt[i] = 3;  
    }  
    for (int i = 1; i <= n; ++i)  
    {  
        if(opt[i] == 1)  
        {  
            if (A > B) ans[i] = 'B', --A, ++B;  
            else if(A < B) ans[i] = 'A', --B, ++A;  
            else  
            {  
                if (opt[i + 1] == 3) ans[i] = 'A', ++A, --B;  
                else ans[i] = 'B', ++B, --A;  
            }  
        }  
        else if(opt[i] == 2)  
        {  
            if (B > C) ans[i] = 'C', ++C, --B;  
            else if(B < C) ans[i] = 'B', ++B, --C;  
            else  
            {  
                if (opt[i + 1] == 1) ans[i] = 'B', ++B, --C;  
                else ans[i] = 'C', ++C, --B;  
            }  
        }  
        else  
        {  
            if (A > C) ans[i] = 'C', ++C, --A;  
            else if (A < C) ans[i] = 'A', ++A, --C;  
            else  
            {  
                if (opt[i + 1] == 1) ans[i] = 'A', ++A, --C;  
                else ans[i] = 'C', ++C, --A;  
            }  
        }  
        if (A < 0 || B < 0 || C < 0) goto fail;  
    }  
    puts("Yes");  
    for (int i = 1; i <= n; ++i) printf("%c\n", ans[i]);  
    return 0;  
fail:  
    puts("No");  
    return 0;  
}  
```