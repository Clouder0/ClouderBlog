---  
title: '树形 DP 练习笔记'  
date: 2020-11-04T10:38:43+08:00  
slug: "dpontreetraining"  
type: posts  
tags:  
  - dynamic-programming  
categories:  
  - Algorithm  
---  
  
  
## 前言  
  
板刷一个[树形 DP 题单](https://www.luogu.com.cn/training/11363#problems)。  
  
## 笔记  
  
动态规划并没有统一的套路，只有部分相似的思想。  
  
### CF219D Choosing Capital for Treeland  
  
有向边，可翻转，要求翻转后以某点为根可走到任一点。  
求最少翻转步数，以及可达到最小步数的根。  
一眼换根 DP，然而只是绿题。  
可以发现，初始以 $1$ 为根，然后假定以 $u$ 为根，有且仅有 $1 \to u$ 的贡献发生变化，那么可以直接枚举计算。  
  
```cpp  
#include <algorithm>  
#include <cstdio>  
using namespace std;  
const int maxn = 1e6 + 100;  
int n;  
struct node  
{  
    int to, val, next;  
} E[maxn];  
int head[maxn];  
inline void add(int x, int y, int z)  
{  
    static int tot = 0;  
    E[++tot].to = y, E[tot].next = head[x], head[x] = tot, E[tot].val = z;  
}  
int dis[maxn], f[maxn], dep[maxn];  
void dfs(int u, int fa)  
{  
    for (int p = head[u]; p; p = E[p].next)  
    {  
        int v = E[p].to;  
        if (v == fa) continue;  
        dep[v] = dep[u] + 1, dis[v] = dis[u] + E[p].val, dfs(v, u), f[u] += f[v] + E[p].val;  
    }  
}  
int main()  
{  
    scanf("%d", &n);  
    for (int i = 1, u, v; i < n; ++i) scanf("%d %d", &u, &v), add(u, v, 0), add(v, u, 1);  
    dfs(1, 0);  
    int minn = f[1];  
    for (int i = 2; i <= n; ++i) minn = min(minn, f[1] - dis[i] + dep[i] - dis[i]);  
    printf("%d\n", minn);  
    for (int i = 1; i <= n; ++i)  
        if (f[1] + dep[i] - 2 * dis[i] == minn) printf("%d ", i);  
    return 0;  
}  
  
```  
  
### [USACO17DEC]Barn Painting G  
  
任意两相邻点颜色不同，那么把颜色加入状态中即可。  
强制某些点选某些颜色，在过程中判断一下更改 DP 值即可。  
普及水平的题目。  
  
```cpp  
#include <algorithm>  
#include <cstdio>  
const int maxn = 1e5 + 100;  
const int mod = 1e9 + 7;  
int n, k, c[maxn];  
struct node  
{  
    int to, next;  
} E[maxn << 1];  
int head[maxn], tot;  
inline void addE(int x, int y) { E[++tot].to = y, E[tot].next = head[x], head[x] = tot; }  
inline int add(int x, int y)  
{  
    int res = x + y;  
    return res >= mod ? res - mod : res;  
}  
inline int mul(int x, int y) { return 1ll * x * y % mod; }  
int f[maxn][4];  
void dfs(int u, int fa)  
{  
    if (c[u]) f[u][c[u]] = 1;  
    else f[u][1] = f[u][2] = f[u][3] = 1;  
    for (int p = head[u]; p; p = E[p].next)  
    {  
        int v = E[p].to;  
        if (v == fa) continue;  
        dfs(v, u);  
        f[u][1] = mul(f[u][1], add(f[v][2], f[v][3]));  
        f[u][2] = mul(f[u][2], add(f[v][1], f[v][3]));  
        f[u][3] = mul(f[u][3], add(f[v][1], f[v][2]));  
    }  
}  
int main()  
{  
    scanf("%d %d", &n, &k);  
    for (int i = 1, u, v; i < n; ++i) scanf("%d %d", &u, &v), addE(u, v), addE(v, u);  
    for (int i = 1, x; i <= k; ++i) scanf("%d", &x), scanf("%d", c + x);  
    dfs(1, 0);  
    printf("%d\n", add(f[1][1], add(f[1][2], f[1][3])));  
    return 0;  
}  
```  
  
### CF461B Appleman and Tree  
  
每个点两个状态：黑/白。  
对于一对父子 $u \to v$，两个选择：切/连。  
  
讨论一下，转移方程自然就出来了。  
  
```cpp  
#include <algorithm>  
#include <cstdio>  
using namespace std;  
const int maxn = 1e5 + 100;  
const int mod = 1e9 + 7;  
struct node  
{  
    int to, next;  
} E[maxn];  
int head[maxn], tot;  
inline void addE(int x, int y) { E[++tot].next = head[x], head[x] = tot, E[tot].to = y; }  
int n, fa[maxn], w[maxn], f[maxn][2];  
inline int add(int x, int y)  
{  
    int res = x + y;  
    return res >= mod ? res - mod : res;  
}  
inline int mul(int x, int y) { return 1ll * x * y % mod; }  
void dfs(int u)  
{  
    f[u][w[u]] = 1;  
    for (int p = head[u]; p; p = E[p].next)  
    {  
        int v = E[p].to;  
        dfs(v);  
        f[u][1] = add(mul(f[u][0], f[v][1]), mul(f[u][1], add(f[v][0], f[v][1])));  
        f[u][0] = mul(f[u][0], add(f[v][0], f[v][1]));  
    }  
}  
int main()  
{  
    scanf("%d", &n);  
    for (int i = 2; i <= n; ++i) scanf("%d", fa + i), ++fa[i], addE(fa[i], i);  
    for (int i = 1; i <= n; ++i) scanf("%d", w + i);  
    dfs(1);  
    printf("%d\n", f[1][1]);  
    return 0;  
}  
  
```  
  
### CF1187E Tree Painting  
  
选定第一个点后，选择方案就是固定的。  
一次 dfs 可以统计出来以某点为根时的情况。  
然后换根。  
  
```cpp  
#include <algorithm>  
#include <cstdio>  
using namespace std;  
const int maxn = 2e5 + 100;  
struct node  
{  
    int to, next;  
} E[maxn << 1];  
int head[maxn], tot;  
inline void add(int x, int y) { E[++tot].next = head[x], head[x] = tot, E[tot].to = y; }  
int n, size[maxn];  
long long f[maxn], g[maxn], k[maxn];  
void dfs1(int u, int fa)  
{  
    size[u] = 1;  
    for (int p = head[u]; p; p = E[p].next)  
    {  
        int v = E[p].to;  
        if (v == fa) continue;  
        dfs1(v, u), size[u] += size[v], f[u] += f[v];  
    }  
    f[u] += size[u];  
}  
void dfs2(int u, int fa)  
{  
    for (int p = head[u]; p; p = E[p].next)  
    {  
        int v = E[p].to;  
        if (v == fa) continue;  
        k[u] = g[u] - f[v] - n + n - size[v];  
        g[v] = f[v] + k[u] - size[v] + n;  
        dfs2(v, u);  
    }  
}  
int main()  
{  
    scanf("%d", &n);  
    for (int i = 1, u, v; i < n; ++i) scanf("%d%d", &u, &v), add(u, v), add(v, u);  
    dfs1(1, 0), g[1] = f[1], dfs2(1, 0);  
    long long maxx = 0;  
    for (int i = 1; i <= n; ++i) maxx = std::max(maxx, g[i]);  
    printf("%lld\n", maxx);  
    return 0;  
}  
```  
  
### [POI2014]MRO-Ant colony  
  
每次向下取整，那么可以对每个叶子结点计算一个可行区间。  
  
```cpp  
#include <algorithm>  
#include <cstdio>  
#include <cctype>  
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
struct node  
{  
    int to, next;  
} E[maxn << 1];  
int head[maxn], tot;  
inline void add(int x, int y) { E[++tot].next = head[x], head[x] = tot, E[tot].to = y; }  
int n, m, k, w[maxn], deg[maxn], maxx;  
long long res;  
long long l[maxn], r[maxn];  
void dfs(int u, int fa)  
{  
	if(l[u] > maxx) return;  
    if (deg[u] == 1)  
    {  
        int ll = std::lower_bound(w + 1, w + 1 + m, l[u]) - w;  
        int rr = std::upper_bound(w + 1, w + 1 + m, r[u]) - w;  
        if (ll < rr) res += 1ll * (rr - ll) * k;  
        return;  
    }  
    for (int p = head[u]; p; p = E[p].next)  
    {  
        int v = E[p].to;  
        if (v == fa) continue;  
        int d = max(1, (deg[v] - 1));  
        l[v] = l[u] * d, r[v] = r[u] * d + d - 1, dfs(v, u);  
    }  
}  
int main()  
{  
    read(n), read(m), read(k);  
    for (int i = 1; i <= m; ++i) read(w[i]);  
    std::sort(w + 1, w + 1 + m);  
    maxx = w[m];  
    int s, t;  
    read(s), read(t), add(s, t), add(t, s), deg[s]++, deg[t]++;  
    for (int i = 2, u, v; i < n; ++i) read(u), read(v), add(u, v), add(v, u), deg[u]++, deg[v]++;  
    int d = max(1, deg[s] - 1);  
    l[s] = 1ll * k * d, r[s] = 1ll * k * d + d - 1;  
    d = max(1, deg[t] - 1), l[t] = 1ll * k * d, r[t] = 1ll * k * d + d - 1;  
    dfs(s, t), dfs(t, s);  
    printf("%lld\n", res);  
    return 0;  
}  
```  
  
### [HEOI2013]SAO  
  
毒瘤题。  
建议查看专门的题解。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <cstring>  
using namespace std;  
const int maxn = 1100;  
int T, n;  
struct node  
{  
    int to, next, val;  
} E[maxn << 1];  
int head[maxn], tot;  
inline void addE(int x, int y, int z) { E[++tot].next = head[x], head[x] = tot, E[tot].to = y, E[tot].val = z; }  
const int mod = 1e9 + 7;  
inline int add(int x, int y) { int res = x + y; return res >= mod ? res - mod : res; }  
inline int mul(int x, int y) { return 1ll * x * y % mod; }  
int C[maxn][maxn];  
int size[maxn], f[maxn][maxn], g[maxn];  
void dfs(int u, int fa)  
{  
    size[u] = 1, f[u][1] = 1;  
    for(int p = head[u]; p; p = E[p].next)  
	{  
		int v = E[p].to;  
        if (v == fa) continue;  
        dfs(v, u);  
        for (int i = 1; i <= size[u]; ++i) g[i] = 0;  
        if (E[p].val)  
        {  
			for(int i = size[u]; i; --i)   
				for(int k = 0; k <= size[v]; ++k)  
					g[i + k] = add(g[i + k],  
						mul(f[v][k],  
						mul(f[u][i],  
						mul(C[i + k - 1][k],  
						C[size[u] + size[v] - i - k][size[v] - k]))));  
		}  
		else  
		{  
			for(int i = size[u]; i; --i)   
				for(int k = 0; k <= size[v]; ++k)  
                    g[i + k] = add(g[i + k],  
                        mul(add(f[v][size[v]], mod - f[v][k]),  
						mul(f[u][i],  
						mul(C[i + k - 1][k],  
						C[size[u] + size[v] - i - k][size[v] - k]))));  
        }  
		size[u] += size[v];  
		for(int i = 1;i<=size[u];++i) f[u][i] = g[i];  
	}  
    for (int i = 1; i <= size[u]; ++i) f[u][i] = add(f[u][i], f[u][i - 1]);  
}  
int main()  
{  
    scanf("%d", &T);  
    C[0][0] = 1;  
    for (int i = 1; i <= 1000; ++i)  
    {  
        C[i][0] = 1;  
        for (int j = 1; j <= 1000; ++j) C[i][j] = add(C[i - 1][j], C[i - 1][j - 1]);  
    }  
    while(T--)  
	{  
        memset(f, 0, sizeof(f));  
        memset(head, 0, sizeof(head));  
        tot = 0;  
        scanf("%d", &n);  
        for (int i = 1, u, v; i < n; ++i)  
        {  
            char opt[3];  
            scanf("%d %s %d", &u, opt, &v);  
            ++u, ++v;  
            if (opt[0] == '<') addE(u, v, 0), addE(v, u, 1);  
            else addE(u, v, 1), addE(v, u, 0);  
        }  
        dfs(1, 0);  
        printf("%d\n", f[1][n]);  
    }  
	return 0;  
}  
```  
  
### [SCOI2015]小凸玩密室  
  
毒瘤题。  
建议查看专门题解。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <cctype>  
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
#define ll long long  
int n, a[maxn], b[maxn], lg[maxn];  
ll f[maxn][19][2], dis[maxn][19];  
int main()  
{  
	read(n);  
    for (int i = 2; i <= n; ++i) lg[i] = lg[i >> 1] + 1;  
    for (int i = 1; i <= n; ++i) read(a[i]);  
    for (int i = 2; i <= n; ++i) read(b[i]);  
    for (int i = 2; i <= n; ++i) for (int j = 1; j <= lg[i]; ++j) dis[i][j] = dis[i >> 1][j - 1] + b[i];  
    for (int i = n; i; --i)  
    {  
        for (int j = 1; j <= lg[i] + 1; ++j)  
        {  
            int p = (i >> (j - 1)) ^ 1;  
            if ((i << 1) > n)  
            {  
                //leaf  
                f[i][j][0] = dis[i][j] * a[i >> j];  
                f[i][j][1] = (dis[i][j] + dis[p][1]) * a[p];  
            }  
			else if((i << 1 | 1) > n)  
			{  
				//only left  
				int ls = i << 1;  
                f[i][j][0] = f[ls][j + 1][0] + dis[ls][1] * a[ls];  
                f[i][j][1] = f[ls][j + 1][1] + dis[ls][1] * a[ls];  
            }  
			else  
			{  
				//both  
                int ls = i << 1, rs = i << 1 | 1;  
                f[i][j][0] = min(f[ls][j + 1][0] + f[rs][1][1] + dis[rs][1] * a[rs], f[rs][j + 1][0] + f[ls][1][1] + dis[ls][1] * a[ls]);  
                f[i][j][1] = min(f[ls][j + 1][1] + f[rs][1][1] + dis[rs][1] * a[rs], f[rs][j + 1][1] + f[ls][1][1] + dis[ls][1] * a[ls]);  
            }  
        }  
    }  
	long long res = 1ll << 62;  
    for (int s = 1; s <= n; ++s)  
    {  
        long long t = f[s][1][0];  
        for (int now = s >> 1, last = s; now > 0; now >>= 1, last >>= 1)  
        {  
            int bro = last ^ 1;  
            if (bro <= n) t += dis[bro][1] * a[bro] + f[bro][2][0];  
            else t += dis[now][1] * a[now >> 1];  
        }  
        res = std::min(res, t);  
    }  
    printf("%lld\n",res);  
    return 0;  
}  
```  
  
### [SWTR-04] Collecting Coins  
  
必须经过 $d$ 点，考虑将这个特殊的点当成根。  
经过次数的限制，可以这样考虑：一个点被访问首先经过一次，然后每访问一棵子树都被经过一次，可以选择 $k - 1$ 个子树。  
  
任意起点，可以考虑换根，那么先做以 $d$ 为根的动态规划。  
  
定义 $f(i)$ 为到达 $i$ 点，在 $d$ 为根的意义下遍历完 $i$ 的子树可以获得的最大贡献。  
  
选择前 $k_i - 1$ 大的子树累加即可更新。  
  
然后考虑换根。  
  
换完根后，多出一个父亲子树，且 $d$ 点一定在父亲子树中。  
  
强制选择父亲子树即可，依然要选择 $k_i - 1$ 个，强制选择父亲子树后就只能选择 $k_i - 2$ 个子树了。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <vector>  
using namespace std;  
const int maxn = 1e5 + 100;  
struct node { int to, next, val; } E[maxn << 1];  
int head[maxn], tot;  
inline void add(int x, int y, int z) { E[++tot].next = head[x], head[x] = tot, E[tot].to = y, E[tot].val = z; }  
int n, d, k[maxn], f[maxn], last[maxn], g[maxn], vis[maxn], res;  
bool rcmp(pair<int,int> a,pair<int,int> b) { return a.first > b.first; }  
vector<pair<int,int> > V[maxn];  
void dfs(int u, int fa)  
{  
	for(int p = head[u], v; p; p = E[p].next)  
	{  
		if((v = E[p].to) == fa) continue;  
		dfs(v, u), V[u].push_back(make_pair(f[v] + E[p].val, v));  
	}  
	sort(V[u].begin(), V[u].end(), rcmp);  
	for(int i = 0; i < (int)V[u].size() && i < k[u] - 1; ++i) f[u] += V[u][i].first;  
}  
void dfs2(int u, int fa, int faval)  
{  
	if(u != d && k[u] == 1) return;  
	g[u] = faval;  
	int up = k[u] - 2 + !fa;  
	for(int i = 0; i < (int)V[u].size() && i < up; ++i) g[u] += V[u][i].first, vis[V[u][i].second] = 1;  
	res = std::max(res, g[u]);  
	int t = 0;  
	if(V[u].size() > up) t = V[u][up].first;  
	for(int p = head[u], v; p; p = E[p].next)  
	{  
		if((v = E[p].to) == fa) continue;  
		if(vis[v]) dfs2(v, u, g[u] - f[v] - E[p].val + t + E[p].val);  
		else dfs2(v, u, g[u] + E[p].val);  
	}  
}  
int main()  
{  
	scanf("%d %d", &n, &d);  
	for(int i = 1, a, b, c; i < n; ++i) scanf("%d %d %d", &a, &b, &c), add(a, b, c), add(b, a, c);  
	for(int i = 1; i <= n; ++i) scanf("%d", k + i);  
	dfs(d, 0), dfs2(d, 0, 0);  
	printf("%d\n",res);  
	return 0;  
}  
```  
  
### [COCI2014-2015#1] Kamp  
  
一次可以载多个人，司机可以不回到起点。  
做一遍树形 DP，然后换根。  
司机一定停留在当前根子树中最长链的端点上，可以节约最多的距离。  
动态维护最长链。  
  
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
const int maxn = 5e5 + 100;  
struct node  
{  
    int to, next, val;  
} E[maxn << 1];  
int head[maxn];  
inline void add(const int& x, const int& y, const int& val)  
{  
    static int tot = 0;  
    E[++tot].next = head[x],E[tot].to = y,E[tot].val = val,head[x] = tot;  
}  
int n, k, point[maxn], size[maxn], iskey[maxn];  
long long f[maxn], maxx[maxn], secx[maxn], g[maxn], ANS[maxn];  
void dfs(int u, int fa)  
{  
    size[u] = iskey[u];  
    for (int p = head[u]; p; p = E[p].next)  
    {  
        int v = E[p].to;  
        if (v == fa) continue;  
        dfs(v, u);   
        if(!size[v]) continue;  
        size[u] += size[v];  
        if (size[v]) f[u] += f[v] + (E[p].val << 1);  
        int len = maxx[v] + E[p].val;  
        if (len >= maxx[u]) secx[u] = maxx[u], maxx[u] = len; // not strictly greater  
        else if (len > secx[u]) secx[u] = len;  
    }  
}  
void dfs2(int u, int fa)  
{  
    ANS[u] = g[u] - maxx[u];  
    for (int p = head[u]; p; p = E[p].next)  
    {  
        int v = E[p].to;  
        if(v == fa) continue;  
        g[v] = f[v];  
        if(k - size[v])   
        {  
            g[v] += g[u] + (E[p].val << 1);  
            if (size[v]) g[v] -= f[v] + (E[p].val << 1);  
            if (maxx[u] > maxx[v] + E[p].val) secx[v] = std::max(secx[u] + E[p].val, maxx[v]), maxx[v] = maxx[u] + E[p].val;  
            else  
            {  
                int len = secx[u] + E[p].val;  
                if (len >= maxx[v]) secx[v] = maxx[v], maxx[v] = len;  
                else if (len > secx[v]) secx[v] = len;  
            }  
        }  
       dfs2(v, u);  
    }  
}  
int main()  
{  
    read(n),read(k);  
    for (int i = 1, a, b, c; i < n; ++i) read(a), read(b), read(c), add(a, b, c), add(b, a, c);  
    for (int i = 1; i <= k; ++i) read(point[i]), iskey[point[i]] = 1;  
    dfs(1, 0), g[1] = f[1], dfs2(1, 0);  
    for (int i = 1; i <= n; ++i) printf("%lld\n", ANS[i]);  
    //puts("");  
    //for (int i = 1; i <= n; ++i) printf("%d %d %d %d %d\n", i, f[i], g[i], maxx[i], secx[i]);  
    return 0;  
}  
```  
  
### [JLOI2016/SHOI2016]侦察守卫  
  
神奇的状态定义。  
  
$f(u,i)$ 代表以 $u$ 为根的子树中，子树中所有关键点已经被覆盖，且还能从 $u$ 点向外覆盖距离至少为 $i$ 的点时的最小代价。  
$g(u,i)$ 代表以 $u$ 为根的子树中，未被覆盖的点离 $u$ 点距离 $< i$ 时的最小代价。  
  
那么对于 $u \to v$，有转移方程：  
  
$f(u,i) = \min(f(v,i + 1) + g(u,i + 1),f(u,i) + g(v,i))$  
  
前半部分是靠 $v$ 往上覆盖，后半部分是靠 $u$ 往下覆盖。  
  
而由定义，往外覆盖 $i + 1$  个点的代价，一定大于等于 $i$ 个的代价，因此有：  
  
$f(u,i) = \min(f(u,i),f(u,i + 1))$  
  
另外一边， $g(u,i) = \sum g(v,i - 1)$  
意义很好理解，不赘述了，同样需要根据定义取 $\min$.  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
const int maxn = 5e5 + 100;  
struct node  
{  
	int to, next;  
}E[maxn << 1];  
int head[maxn], tot;  
inline void add(int x, int y){ E[++tot].next = head[x], head[x] = tot, E[tot].to = y; }  
int n, m, d, w[maxn], f[maxn][21], g[maxn][21], iskey[maxn];  
const int inf = 1 << 29;  
void dfs(int u, int fa)  
{  
	if(iskey[u]) f[u][0] = g[u][0] = w[u];  
	for(int i = 1;i <= d; ++i) f[u][i] = w[u];  
	f[u][d + 1] = g[u][d + 1] = inf;  
	for(int p = head[u]; p; p = E[p].next)  
	{  
		int v = E[p].to;  
		if(v == fa) continue;  
		dfs(v, u);  
		for(int i = d; i >= 0; --i) f[u][i] = min(f[v][i + 1] + g[u][i + 1],f[u][i] + g[v][i]);  
		for(int i = d; i >= 0; --i) f[u][i] = min(f[u][i + 1], f[u][i]);  
		g[u][0] = f[u][0];  
		for(int i = 1; i <= d; ++i) g[u][i] += g[v][i - 1];  
		for(int i = 1; i <= d; ++i) g[u][i] = min(g[u][i - 1], g[u][i]);  
	}  
  
}  
int main()  
{  
	scanf("%d %d", &n, &d);  
	for(int i = 1;i <= n;++i) scanf("%d", w + i);  
	scanf("%d", &m);  
	for(int i = 1, x; i <= m; ++i) scanf("%d", &x), iskey[x] = 1;  
	for(int i = 1, u, v; i < n; ++i) scanf("%d %d", &u, &v), add(u, v), add(v, u);  
	dfs(1, 0);  
	printf("%d\n", g[1][0]);  
	return 0;  
}  
```  
  
### [SDOI2011]消防  
  
可以发现，选择的路径一定在直径上。  
否则假定有一个拐点，那么拐点往外，直径上的链一定是最长的。  
考虑找出直径，然后在直径上，移动选择区间。  
计算直径上每个点向外挂的最长链长度，然后计算选择区间中外挂最长链的长度，与区间到两端的值取 $\max$ 计算答案。  
  
类似滑动窗口单调队列的维护方法。  
  
```cpp  
#include <algorithm>  
#include <cstdio>  
#include <cctype>  
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
const int maxn = 3e5 + 100;  
struct node  
{  
    int to, next, val;  
} E[maxn << 1];  
int head[maxn], tot;  
inline void add(int x, int y, int z) { E[++tot].next = head[x], head[x] = tot, E[tot].to = y, E[tot].val = z; }  
int n, s;  
int dis[maxn], pre[maxn];  
void dfs(int u, int fa)  
{  
    for (int p = head[u]; p; p = E[p].next)  
    {  
        int v = E[p].to;  
        if (v == fa) continue;  
        dis[v] = dis[u] + E[p].val, pre[v] = u, dfs(v, u);  
    }  
}  
int ondia[maxn], dia[maxn], cnt;  
int maxdis[maxn], vis[maxn], bel[maxn], q[maxn], qt, qh;  
int main()  
{  
    read(n), read(s);  
    for (int i = 1, a, b, c; i < n; ++i) read(a), read(b), read(c), add(a, b, c), add(b, a, c);  
    dfs(1, 0);  
    int from = 1, to = 1;  
    for (int i = 1; i <= n; ++i) if (dis[from] < dis[i]) from = i;  
    dis[from] = 0, dfs(from, 0);  
    for (int i = 1; i <= n; ++i) if (dis[to] < dis[i]) to = i;  
    while (from != to) dia[++cnt] = to, to = pre[to];  
    dia[++cnt] = from;  
    reverse(dia + 1, dia + 1 + cnt);  
    for (int i = 1; i <= cnt; ++i) ondia[dia[i]] = 1, q[++qt] = dia[i], bel[dia[i]] = dia[i];  
    qh = 1;  
    while (qt >= qh)  
    {  
        int u = q[qh++];  
        for (int p = head[u]; p; p = E[p].next)  
        {  
            int v = E[p].to;  
            if (ondia[v] || vis[v]) continue;  
            vis[v] = 1, maxdis[v] = maxdis[u] + E[p].val, bel[v] = bel[u];  
            q[++qt] = v;  
        }  
    }  
    for (int i = 1; i <= n; ++i) if (!ondia[i]) maxdis[bel[i]] = max(maxdis[bel[i]], maxdis[i]);  
    qh = 1, qt = 0;  
    int r = 0;  
    int res = 1 << 30;  
    for (int l = 1; l <= cnt; ++l)  
    {  
        while (r < cnt && dis[dia[r + 1]] - dis[dia[l]] <= s)  
        {  
            ++r;  
            while (qt >= qh && maxdis[dia[q[qt]]] <= maxdis[dia[r]]) --qt;  
            q[++qt] = r;  
        }  
        while (qt >= qh && q[qh] < l) ++qh;  
        int left = dis[dia[l]], right = dis[dia[cnt]] - dis[dia[r]];  
        int now = max(left, right);  
        if (qt >= qh) now = max(now, maxdis[dia[q[qh]]]);  
        res = min(res, now);  
    }  
    printf("%d\n", res);  
    return 0;  
}  
```