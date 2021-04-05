---  
Title: '[OI题库]八月提高模拟题解'  
date: 2020-09-03T18:33:07+08:00  
slug: "oitiku8mon"  
type: posts  
tags:  
categories:  
  - Algorithm  
---  
  
## suffix  
  
### 题意  
  
如题面所述。  
  
### 解法  
  
#### 10pts  
  
暴力做法即可。  
  
#### 50pts  
  
考虑链的情况。  
  
对于当前区间 $[1,r]$，若 $i$ 位置的数后无比它大的数，则可计入贡献。  
  
顺着遍历，维护单调不增栈，答案即为栈的大小。  
  
对于栈中每点，都保证了它后面无比它大的数，否则它将被弹出。  
  
时间复杂度 $O(n)$.  
  
```cpp  
#include <cstdio>  
const int maxn = 1e6 + 100;  
int n,fa[maxn],w[maxn];  
int st[maxn],top;  
int main()  
{  
    scanf("%d",&n);  
    for (int i = 2; i <= n; ++i) scanf("%d", fa + i);  
    for (int i = 1; i <= n; ++i) scanf("%d", w + i);  
    for (int i = 1; i <= n; ++i)  
    {  
        while (top && w[st[top]] < w[i]) --top;  
        st[++top] = i;  
        printf("%d ", top);  
    }  
    return 0;  
}  
```  
  
#### 70pts  
  
考虑将链的做法拓展到树上。  
  
在遍历到树上某点的时候，得出根节点到该点的链形成的单调栈。  
  
在回溯的过程中，撤销对单调栈的更改。  
  
具体地，将该点插入单调栈时，只会改变栈顶位置和插入点的值。记录原栈顶位置、插入点更改前的值，即可在回溯时撤销。  
  
```cpp  
void dfs(int u)  
{  
    int oldtop = top, oldval = 0;  
    while (top && w[st[top]] < w[u]) --top;  
    oldval = st[++top], st[top] = u;  
    ans[u] = top;  
    for (int p = head[u]; p; p = E[p].next) dfs(E[p].to);  
    st[top] = oldval, top = oldtop;  
}  
```  
  
#### 100pts  
  
上面的这种做法，最劣复杂度为 $O(n^2)$.  
  
考虑一条长为 $\dfrac{n}{2}$ 的链，链端接 $\dfrac{n}{2}$ 个点，每个点都要求将单调栈弹空，此时朴素做法肯定会超时。  
  
使用二分的方法找新的栈顶即可通过。  
  
时间复杂度 $O(n \log n)$.  
  
```cpp  
#include <cstdio>  
const int maxn = 1e6 + 100;  
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
int n, fa[maxn], w[maxn];  
int st[maxn], top;  
int ans[maxn];  
void dfs(int u)  
{  
    int oldtop = top, oldval = 0;  
    int l = 1, r = top, mid, res = 0;  
    while (l <= r)  
    {  
        mid = l + r >> 1;  
        if (w[st[mid]] >= w[u]) res = mid, l = mid + 1;  
        else r = mid - 1;  
    }  
    top = res + 1, oldval = st[top], st[top] = u;  
    ans[u] = top;  
    for (int p = head[u]; p; p = E[p].next) dfs(E[p].to);  
    st[top] = oldval, top = oldtop;  
}  
int main()  
{  
    scanf("%d",&n);  
    for (int i = 2; i <= n; ++i) scanf("%d", fa + i), add(fa[i], i);  
    for (int i = 1; i <= n; ++i) scanf("%d", w + i);  
    dfs(1);  
    for (int i = 1; i <= n; ++i) printf("%d ", ans[i]);  
    return 0;  
}  
```  
  
## Interval  
  
### 题意  
  
求满足最小值与最大值和为定值的区间个数。  
  
### 解法  
  
#### 30pts  
  
考虑暴力做法，枚举左右端点，遍历区间得到最大最小值，检验是否合法。复杂度 $O(n^3)$.  
  
在枚举右端点时，可以直接更新最大最小值，复杂度优化为 $O(n^2)$.  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
const int maxn = 1e6 + 100;  
int n, x, a[maxn];  
int main()  
{  
    scanf("%d %d", &n, &x);  
    for (int i = 1; i <= n; ++i) scanf("%d", a + i);  
    long long ans = 0;  
    for (int l = 1; l <= n; ++l)  
    {  
        int maxx = a[l], minn = a[l];  
        for (int r = l; r <= n; ++r)  
        {  
            maxx = std::max(maxx, a[r]), minn = std::min(minn, a[r]);  
            if (maxx + minn == x) ++ans;  
        }  
    }  
    printf("%lld\n", ans);  
    return 0;  
}  
```  
  
#### 60pts  
  
想拿到随机数据的分有很多种办法。  
  
可以发现，在上述 $O(n^2)$ 的算法中，有许多右端点移动是无效的，即不会更新最大最小值。  
  
如果优化掉这部分无效移动，就能显著提升算法效率。  
  
使用单调栈预处理出每个位置向右第一个严格大于、严格小于它的数的位置。  
  
记录当前区间最大、最小值的位置，然后每次直接跳到下一个能更新最大、最小值的位置即可。  
  
该算法在随机数据下表现良好，可以拿到 90pts 的好成绩。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
const int maxn = 1e6 + 100;  
int n, x, a[maxn];  
int rmax[maxn], rmin[maxn];  
int st[maxn], top;  
int main()  
{  
    scanf("%d %d", &n, &x);  
    for (int i = 1; i <= n; ++i) scanf("%d", a + i);  
    for (int i = 1; i <= n; ++i)  
    {  
        while (top && a[st[top]] > a[i]) rmin[st[top--]] = i;  
        st[++top] = i;  
    }  
    while (top) rmin[st[top--]] = n + 1;  
    for (int i = 1; i <= n; ++i)  
    {  
        while (top && a[st[top]] < a[i]) rmax[st[top--]] = i;  
        st[++top] = i;  
    }  
    while (top) rmax[st[top--]] = n + 1;  
    long long ans = 0;  
    for (int l = 1; l <= n; ++l)  
    {  
        int maxx = l, minn = l;  
        int r = l;  
        while (r <= n)  
        {  
            if (a[maxx] > x) break;  
            if (a[minn] + a[maxx] == x) ans += std::min(rmin[minn], rmax[maxx]) - r;  
            r = std::min(rmin[minn], rmax[maxx]);  
            if (a[r] > a[maxx]) maxx = r;  
            if (a[r] < a[minn]) minn = r;  
        }  
    }  
    printf("%lld\n", ans);  
    return 0;  
}  
```  
  
#### 100pts  
  
考虑枚举取到最大值的点 $i$，则 $a_i$ 为最大值的区间可以用单调栈计算出来。  
  
特判 $a_i$ 同时为最大、最小值的情况方便后续处理。  
  
左端点可位于 $[L,i-1]$ 中，右端点可位于 $[i+1,R]$ 中。  
  
此时需要的最小值为 $x - a_i$，记为 $need$.  
  
考虑计算出从 $i$ 点向左多远、向右多远可以取到 $need$，而多远后不再能取到 $need$.  
  
最小值单调递减，因此可以使用二分计算。使用 ST 表预处理，可以保证二分复杂度为 $O(\log n)$.  
  
当左端点位于 $[L_1,R_1]$ 中时，向左最小值可以取到 $need$，而当右端点位于 $[L_2,R_2]$ 中时，向右最小值可以取到 $need$.  
  
当左端点位于 $[L_1,R_1]$ 中时，右端点可以取 $[i,R_2]$.  
  
当右端点位于 $[L_2,R_2]$ 中时，左端点可以取 $[L_1,i]$.  
  
用乘法原理统计，再减去重复计算的部分，即左端点在 $[L_1,R_1]$ 中，右端点在 $[L_2,R_2]$ 中的情况。  
  
还要注意一个细节，即最大值区间内有重复数字时可能重复计算，因此区间取左边第一个大于它的数与右边第一个大于等于它的数。  
  
总复杂度 $O(n \log n)$，常数巨大，但可以通过。  
  
```cpp  
#include <algorithm>  
#include <cstdio>  
  
const int maxn = 1e6 + 100;  
int n, x, a[maxn];  
int minn[maxn][22], lg[maxn];  
  
void STtableInit()  
{  
    for (int i = 2; i <= n; ++i) lg[i] = lg[i / 2] + 1;  
    for (int i = 1; i <= n; ++i) minn[i][0] = a[i];  
    for (int j = 1; j <= lg[n]; ++j)  
        for (int i = 1; i + (1 << j) - 1 <= n; ++i)  
            minn[i][j] = std::min(minn[i][j - 1], minn[i + (1 << (j - 1))][j - 1]);  
}  
inline int askmin(int l, int r)  
{  
    int k = lg[r - l + 1];  
    return std::min(minn[l][k], minn[r - (1 << k) + 1][k]);  
}  
  
int lmax[maxn], rmax[maxn], lmin[maxn], rmin[maxn];  
int st[maxn], top;  
inline void stackinit()  
{  
    for (int i = 1; i <= n; ++i)  
    {  
        while (top && a[st[top]] <= a[i]) rmax[st[top--]] = i;  
        lmax[i] = st[top], st[++top] = i;  
    }  
    while (top) rmax[st[top--]] = n + 1;  
    for (int i = 1; i <= n; ++i)  
    {  
        while (top && a[st[top]] >= a[i]) --top;  
        lmin[i] = st[top], st[++top] = i;  
    }  
    st[top = 0] = n + 1;  
    for (int i = n; i; --i)  
    {  
        while (top && a[st[top]] >= a[i]) --top;  
        rmin[i] = st[top], st[++top] = i;  
    }  
}  
  
int main()  
{  
    scanf("%d %d", &n, &x);  
    for (int i = 1; i <= n; ++i) scanf("%d", a + i);  
    STtableInit(), stackinit();  
    long long res = 0;  
    for (int i = 1; i <= n; ++i)  
    {  
        if (a[i] > x) continue;  
        int need = x - a[i], L = lmax[i] + 1, R = rmax[i] - 1;  
        if (need > a[i]) continue;  
        if (need == a[i])  
        {  
            L = std::max(L, lmin[i] + 1), R = std::min(R, rmin[i] - 1);  
            res += 1ll * ((i - 1) - L + 1) * (R - (i + 1) + 1);  
            res += (i - 1) - L + 1;  
            res += R - (i + 1) + 1;  
            res++;  
            continue;  
        }  
        int l1, r1, l2, r2;  
        int t, l, r, mid, ans, can;  
  
        l = L, r = i - 1, ans = -1, can = i;  
        while (l <= r)  
        {  
            mid = l + r >> 1;  
            t = askmin(mid, i);  
            if (t > need) can = mid, r = mid - 1;  
            else if (t < need) l = mid + 1;  
            else ans = mid, l = mid + 1;  
        }  
        r1 = ans;  
        if (r1 != -1) l1 = std::max(L, lmin[r1] + 1); else l1 = can;  
  
        l = i + 1, r = R, ans = -1, can = i;  
        while (l <= r)  
        {  
            mid = l + r >> 1;  
            t = askmin(i, mid);  
            if (t > need) can = mid, l = mid + 1;  
            else if (t < need) r = mid - 1;  
            else ans = mid, r = mid - 1;  
        }  
        l2 = ans;  
        if (l2 != -1) r2 = std::min(R, rmin[l2] - 1); else r2 = can;  
  
        if (r1 != -1) res += 1ll * (r1 - l1 + 1) * (r2 - i + 1);  
        if (l2 != -1) res += 1ll * (r2 - l2 + 1) * (i - l1 + 1);  
        if (r1 != -1 && l2 != -1) res -= 1ll * (r1 - l1 + 1) * (r2 - l2 + 1);  
    }  
    printf("%lld\n", res);  
    return 0;  
}  
```  
  
## Block  
  
### 题意  
  
给定询问，求所有块长下，分块的运行复杂度。  
  
需要注意，当左右端点与块的左右端点重合时，不作为整块处理。  
  
### 解法  
  
#### 30pts  
  
暴力，枚举块长，对每个块长，遍历所有询问，计算对应的复杂度。  
  
时间复杂度 $O(nm)$.  
  
#### 90pts  
  
做法大题与 100pts 相同，但使用整除分块。  
  
#### 100pts  
  
考虑枚举块长，对块长，枚举所有块。  
  
这样复杂度是 $\sum \limits _{i=1}^n \lfloor \dfrac{n}{i} \rfloor$，调和级数，近似为 $O(n \log n)$.  
  
考虑已知块长时，如何通过枚举块计算出答案。  
  
若一个询问包含当前块，则可以节省 $B - 1$ 次运算。  
  
对每一个枚举的块，计算出有多少个询问包含它即可。  
  
考虑如何计算块被多少个询问包含。  
  
一个询问 $l_i < L$ 且 $r_i > R$ 时，包含该块。  
  
$r_i \leq R$ 的询问中，分为 $l_i < L$ 与 $l_i \geq L$ 两类。  
  
当 $l_i \geq L$ 且 $r_i \leq R$ 时，该询问完全在块内，即 $r_i - l_i + 1 \leq B$.  
  
只要除去完全在块内的询问，计算 $l_i < L$ 的询问数与 $r_i \leq R$ 的询问数，用前者减去后者即可得出块被多少询问包含。可以用树状数组来处理。  
  
除去完全在块内的询问，可以转化为：当询问区间长度大于块长时，才插入树状数组中。  
  
给询问排序，利用单调性即可。  
  
时间复杂度 $O(n \log ^2 n)$，常数较小，可以通过。  
  
```cpp  
#include <algorithm>  
#include <cstdio>  
#include <ctype.h>  
const int bufSize = 1e6;  
inline char nc()  
{  
    static char buf[bufSize], *p1 = buf, *p2 = buf;  
    return p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, bufSize, stdin), p1 == p2) ? EOF : *p1++;  
}  
template <typename T>  
inline T read(T& r)  
{  
    static char c; r = 0;  
    for (c = nc(); !isdigit(c); c = nc()) ;  
    for (; isdigit(c); c = nc()) r = r * 10 + c - 48;  
    return r;  
}  
const int maxn = 1e6 + 100;  
int n, m, p = 1;  
long long ans[maxn], sum;  
struct query { int l, r, len; } Q[maxn];  
bool cmp(const query& a, const query& b) { return a.len > b.len; }  
struct bit  
{  
    int sum[maxn];  
    inline int ask(int x)  
    {  
        int res = 0;  
        for (; x > 0; x -= (x & -x)) res += sum[x];  
        return res;  
    }  
    inline void add(int x, int k) { for (; x <= n; x += (x & -x)) sum[x] += k; }  
} T1, T2;  
int main()  
{  
    read(n), read(m);  
    for (int i = 1; i <= m; ++i) read(Q[i].l), read(Q[i].r), sum += Q[i].len = Q[i].r - Q[i].l + 1;  
    std::sort(Q + 1, Q + 1 + m, cmp);  
    for (int B = n; B > 1; --B)  
    {  
        while (p <= m && Q[p].len > B) T1.add(Q[p].l, 1), T2.add(Q[p].r, 1), ++p;  
        long long num = 0;  
        for (int l = B + 1; l <= n; l += B)  
        {  
            int r = std::min(n, l + B - 1);  
            num += T1.ask(l - 1) - T2.ask(r);  
        }  
        ans[B] = sum - 1ll * num * (B - 1);  
    }  
    printf("%lld ", sum);  
    for (int i = 2; i <= n; ++i) printf("%lld ", ans[i]);  
    return 0;  
}  
```