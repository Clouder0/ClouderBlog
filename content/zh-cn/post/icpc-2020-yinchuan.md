---  
title: 'ICPC 2020 银川站'
date: 2022-11-10T13:33:05+08:00
slug: "ccpc-2021-harbin"  
type: post  
tags:  
  - icpc
categories:  
  - Algorithm  
---  

清早（指 9 点）拉上队友 VP，起床真是困难至极啊。

人生第一把虚拟 Au，值得记录。虽然这个站人多队水，很是神奇，6 题 Au 的黄金时代。

打完还剩一个多小时，刚好到饭点。

## A

A 题居然是货真价实的签到题。把某维坐标改成零然后记不同的个数就行了。

```cpp
#include <algorithm>
#include <cstdio>
struct node
{
    int x, y, z;
    bool operator<(const node& other)
    {
        if (x != other.x) return x < other.x;
        if (y != other.y) return y < other.y;
        return z < other.z;
    }
} A[110], B[110];
int n;
int main()
{
    scanf("%d", &n);
    for (int i = 1; i <= n; ++i) scanf("%d%d%d", &A[i].x, &A[i].y, &A[i].z), B[i] = A[i];
    for (int i = 1; i <= n; ++i) B[i].x = 0;
    std::sort(B + 1, B + 1 + n);
    int res = 1, resi = 1, now = 1;
    for (int i = 2; i <= n; ++i)
    {
        if (!(B[i].x == B[i - 1].x && B[i].y == B[i - 1].y && B[i].z == B[i - 1].z)) ++now;
    }
    if (now > res) resi = 1, res = now;
    for (int i = 1; i <= n; ++i) B[i] = A[i], B[i].y = 0;
    std::sort(B + 1, B + 1 + n);
    now = 1;
    for (int i = 2; i <= n; ++i)
        if (!(B[i].x == B[i - 1].x && B[i].y == B[i - 1].y && B[i].z == B[i - 1].z)) ++now;
    if (now > res) resi = 2, res = now;
    for (int i = 1; i <= n; ++i) B[i] = A[i], B[i].z = 0;
    std::sort(B + 1, B + 1 + n);
    now = 1;
    for (int i = 2; i <= n; ++i)
        if (!(B[i].x == B[i - 1].x && B[i].y == B[i - 1].y && B[i].z == B[i - 1].z)) ++now;
    if (now > res) resi = 3, res = now;
    if (resi == 1) puts("X");
    else if (resi == 2) puts("Y");
    else puts("Z");
    return 0;
}
```

## E

签到题。高中化学知识。按着题意来判断即可。一开始因为 scanf 没有清空旧的字符串(果然只加一个 `\0`​ 真是坏文明啊）而莫名其妙挂了好几发。

```cpp
#include <cstdio>
#include <cstring>

char s[110];
int T, r1, r2, r3, r4;

char ss[10][100] = {"", "-F", "-Cl", "-Br", "-I", "-CH3", "-CH2CH3", "-CH2CH2CH3", "-H"};

int parse(char *s)
{
    for (int i = 1; i <= 8; ++i)
        if (strcmp(s, ss[i]) == 0) return i;
    return 8;
}

int sgn(int x) { return x > 0 ? 1 : -1; }

int main()
{
    scanf("%d",&T);
    while(T--)
    {
        scanf("%s", s);
        r1 = parse(s);
        scanf("%s", s);
        r2 = parse(s);
        scanf("%s", s);
        r3 = parse(s);
        scanf("%s", s);
        r4 = parse(s);
        if(r1 == r3 || r2 == r4) goto none;
        if(r1 == r2 || r3 == r4) goto cis;
        if(r1 == r4 || r2 == r3) goto trans;
        if (sgn(r1 - r3) == sgn(r2 - r4)) goto zasa;
        goto en;
    none:
        puts("None");
        continue;
    cis:
        puts("Cis");
        continue;
    trans:
        puts("Trans");
        continue;
    zasa:
        puts("Zasamman");
        continue;
    en:
        puts("Entgegen");
        continue;
    }
    return 0;
}
```

## J

一个简单的模拟题。给出每个点的四向边，还原矩阵。

可以考虑先找到左上角，然后依次处理即可。

注意一个特殊情况：只有一个点的时候。

```cpp
#include <cstdio>
const int maxn = 1e3 + 100;
int n;
int to[maxn * maxn][4];
int main()
{
    scanf("%d", &n);
    int start = 0;
    for (int i = 1; i <= n * n; ++i)
    {
        for (int j = 0; j < 4; ++j) scanf("%d", to[i] + j);
        if(to[i][0] == -1 && to[i][2] == -1) start = i;
    }
    if(n == 1) 
    {
        puts("1");
        return 0;
    }
    for (int i = 1; i <= n; ++i)
    {
        printf("%d ", start);
        int p = to[start][3];
        for (int j = 2; j < n; ++j)
        {
            printf("%d ", p);
            p = to[p][3];
        }
        printf("%d", p);
        start = to[start][1];
        if(i != n) puts("");
    }
    return 0;
}
```

## K

中档字符串题。

现在有一堆 URL，依次从 $1$ 到 $n$ 发布 URL，然后有一个类似防火墙规则的东西：只允许访问拥有「允许前缀」的 URL. 要求未发布的 URL 不能访问。

问每发布一个 URL 后的最少允许前缀个数。

在 Trie 上做插入，先把所有的未发布插入打上 tag，然后每次发布就撤销 tag. 如果某个点没有被 tag 标记，说明从根到该点的前缀不会允许未发布的 URL.

那么每次发布一个 URL 后，在 Trie 上遍历这个字符串，更新 tag，然后在从上到下第一个可以作为「允许前缀」的点设置允许前缀。可以发现，在合法情况下，允许前缀一定是越短越优的。并且允许前缀的点随着 URL 的发布而不断上移。

每次设置一个允许点后，其子树内的所有允许点都不再被需要，利用这个性质做一个计数就好了。答案就是全树的允许点个数。

```cpp
#include <cstdio>
#include <cstring>
const int maxn = 3e6 + 100;
int n;
char n2c[28], c2n[1000];
int res;
struct Trie
{
    int fa[maxn], to[maxn][28], cnt, untag[maxn], sum[maxn];
    void ins_rel(char* s)
    {
        int p = 0, place = -1;
        untag[p]--;
        for (; *s != '\0'; ++s)
        {
            int c = c2n[(int)*s];
            if (!to[p][c]) to[p][c] = ++cnt, fa[cnt] = p;  // create a new edge to that node
            p = to[p][c];
            untag[p]--;
            if (untag[p] == 0 && place == -1) place = p;
        }
        int delta = 1 - sum[place];
        res = res + delta;
        while (place != 0) sum[place] += delta, place = fa[place];
    }
    void ins_un(char* s)
    {
        int p = 0;
        untag[p]++;
        for (; *s != '\0'; ++s)
        {
            int c = c2n[(int)*s];
            if (!to[p][c]) to[p][c] = ++cnt, fa[cnt] = p;  // create a new edge to that node
            p = to[p][c];
            untag[p]++;
        }
    }
} T;
char s[51000][51];
int main()
{
    for (int i = 0; i < 26; ++i) n2c[i] = 'a' + i, c2n['a' + i] = i;
    n2c[26] = '.', n2c[27] = '/';
    c2n['.'] = 26, c2n['/'] = 27;
    scanf("%d", &n);
    for (int i = 1; i <= n; ++i)
        scanf("%s", s[i] + 1), T.ins_un(s[i] + 1);
    for (int i = 1; i <= n; ++i)
    {
        T.ins_rel(s[i] + 1);
        printf("%d\n", res);
    }
    return 0;
}
```

## B

中档 DP 题。郑神单挑的题目。

将长度为 $n$ 的序列进行划分，每一段区间的贡献为区间内最大值减去最小值。问划分为 $1,2,\cdots,n$ 段时的贡献最大值。

这里的特殊之处在于最大值减最小值可以转化为区间内任意两个数的差值的最大值，这一点性质是非常关键的。

定义 $f(i,j,0)$ 为考虑前 $i$ 个数，划分为 $j$ 段，且最后一段尚未选择数的情况下的最大贡献。

$f(i,j,1)$ 为只选择了被减数的最大贡献。

$f(i,j,2)$ 为只选择了减数的最大贡献。

$f(i,j,3)$ 为减数和被减数都选择后的最大贡献。

然后朴素转移即可。滚动数组把空间处理好就行。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;
const int maxn = 1e4 + 100;
int n, a[maxn];
int f[2][maxn][4];
int main()
{
    scanf("%d", &n);
    for (int i = 1; i <= n; ++i) scanf("%d", a + i);
    int now = 0, last = 1;
    memset(f, -0x3f, sizeof(f));
    f[last][0][3] = 0;
    for (int i = 1; i <= n; ++i)
    {
        for (int j = 1; j <= i; ++j)
        {
            f[now][j][0] = max(f[last][j][0], f[last][j - 1][3]);
            f[now][j][1] = max(f[last][j - 1][3] + a[i], max(f[last][j][0] + a[i], f[last][j][1]));
            f[now][j][2] = max(f[last][j - 1][3] - a[i], max(f[last][j][0] - a[i], f[last][j][2]));
            f[now][j][3] = max(f[last][j][1] - a[i], max(f[last][j][2] + a[i], max(f[last][j][3], f[last][j - 1][3])));
        }
        last ^= 1, now ^= 1;
    }
    for (int i = 1; i <= n; ++i) printf("%d\n", f[last][i][3]);
    return 0;
}
```

## G

中档思维数据结构题。什么叫做连罚 7 发的含金量啊……

有一个很朴素的 $O(qn\log n)$ 的做法，算出来大概 $2 \times 10^8$，卡了好几发常都没卡过去，无奈放弃了。

直接模拟是没有前途的，这道题的恶心之处就在于误导你往朴素模拟这个简单又仿佛差一点点就能卡过的思路想。

事实上直接考虑计算总答案。

对于一个编号为 $i$ 的人，他在 $t(i)$ 时刻被加入。那么这个人左侧将是什么人呢？可以发现，他的左侧的人是谁应该是随着时间变化的。

编号越大、加入时间越小的人，越有可能成为左侧相邻者。

那我们维护一个单调栈，编号、时间递增。

与当前比较，如果 $t(i) > t(top)$，那么这个数插入之后，栈顶元素就一直是他的左侧相邻人。

否则的话，就弹栈，并且可以发现在这个过程中， 栈顶会当「一段时间」他的左侧相邻人。这段时间就是从栈顶加入到旧栈顶加入之间的时间，$[t(top),last)$，初始令 $last = n + 1$ 即可。最后又回到了第一种情况。

然后统计答案。$O(qn)$.

```cpp
#include <cstdio>
#include <set>
#include <algorithm>
#include <cctype>
using namespace std;
const int maxn = 1e6 + 100;
#define int long long
int n, q, h[maxn], p[maxn], k, np[maxn];
long long ans;
int sav[11000];
template <typename T>
inline void read(T& r)
{
    r = 0;
    static char c;
    c = getchar();
    for (; !isdigit(c);) c = getchar();
    for (; isdigit(c); c = getchar()) r = r * 10 + c - 48;
}
void solve()
{
    ans = 0;
    long long nowans = 0;
    set<int> S;
    S.insert(p[1]);
    for (int i = 2; i <= n; ++i)
    {
        int now = p[i];
        if (S.find(now) != S.end()) 
        {
            ans += nowans;
            continue;
        }
        S.insert(now);
        set<int>::iterator it = S.find(now);
        set<int>::iterator pre = it;
        set<int>::iterator nxt = it;
        ++nxt;
        if (it == S.begin())
        {
            nowans += sav[abs(h[now] - h[*nxt])];
            goto end;
        }
        --pre;
        if (nxt == S.end())
        {
            nowans += sav[abs(h[now] - h[*pre])];
            goto end;
        }
        nowans = nowans - sav[abs(h[*nxt] - h[*pre])] + sav[abs(h[now] - h[*nxt])] + sav[abs(h[now] - h[*pre])];
    end:
        ans += nowans;
    }
}
int s[maxn], top, t[maxn];
inline int calc(int a, int b) { return sav[abs(h[a] - h[b])]; }
void solve2()
{
    for (int i = 1; i <= n; ++i) t[p[i]] = i;
    ans = 0;
    top = 0;
    s[++top] = 1;
    for (int i = 2; i <= n; ++i)
    {
        int last = n + 1;
        while (top && t[s[top]] > t[i]) 
            ans += 1ll * calc(i, s[top]) * (last - t[s[top]]), last = t[s[top]], --top;
        if (top) ans += 1ll * calc(i, s[top]) * (last - t[i]);
        s[++top] = i;
    }
}
signed main()
{
    read(n), read(q);
    for (int i = 1; i <= n; ++i) read(h[i]);
    for (int i = 1; i <= n; ++i) read(p[i]);
    for (int i = 0; i <= 10001; ++i) sav[i] = i * i;
    solve2();
    printf("%lld\n", ans);
    for (int i = 1; i <= q; ++i)
    {
        read(k);
        k += ans;
        k %= n;
        for (int j = 1; j + k <= n; ++j) np[j] = p[j + k];
        for (int j = 1; j <= k; ++j) np[n - k + j] = p[j];
        for (int j = 1; j <= n; ++j) p[j] = np[j];
//        puts("!!!");
//        for(int j = 1;j<=n;++j) printf("%lld ",p[i]);
//        puts("");
//        puts("!!!");
        solve2();
        printf("%lld\n", ans);
    }
    return 0;
}
```
