---
title: 'NOI 系列真题练习笔记'
date: 2020-11-20T08:35:02+08:00
slug: "noiptrain"
type: posts
tags:
  - noip
  - csp
categories:
  - Algorithm
---

## 前言

NOIP 前开始做做真题，虽然每年都风格迥异，不过感受一下 OI 风格的题目还是有一定意义的。

### P1016 旅行家的预算

贪心地考虑，在一个油站，如果加油，要么加满，要么加到足够行驶到下一个加油点的油量，在下一个更便宜的加油点再加油。

是否存在后效性呢？在下一个油站始终可以加满，所以应该没有。
当然，这鬼数据范围乱搜一下就完事了。

```cpp
#include <cstdio>
const int maxn = 100;
int n;
double totaldis,C,per;
double cost[maxn],dis[maxn];
double res = 1e18;
void dfs(int now,double totalcost,double nowhave)
{
	if(nowhave < 0) return;
	if(now == n) return (void)(res = res > totalcost ? totalcost : res);
	for(int i = now + 1;i<=n;++i)
	{
		double d = dis[i] - dis[now];
		if(cost[i] <= cost[now] && nowhave * per >= d) { dfs(i,totalcost,nowhave - d / per); return; }
	}
	//if no better choice can be reached without cost, then should choose one that is cheaper and tries to go there
	bool flag = 0;
	for(int i = now + 1;i<=n;++i) 
	{
		double d = dis[i] - dis[now];
		if(cost[i] <= cost[now])
		{
			double needoil = (d - nowhave * per) / per;
			if(nowhave + needoil > C) continue;
			double need = needoil * cost[now];
			dfs(i,totalcost + need,0);
			flag = 1;
		}
	}
	if(flag) return;
	//if no cheaper station, add here to full and tries to go.
	for(int i = now + 1;i<=n;++i)
	{
		double d = dis[i] - dis[now];
		if(cost[i] > cost[now])
		{
			double need = (C - nowhave) * cost[now];
			dfs(i,totalcost + need,C - d / per);
		}
	
	}
}
int main()
{
	scanf("%lf %lf %lf %lf %d",&totaldis,&C,&per,cost,&n);
	for(int i = 1;i<=n;++i) scanf("%lf %lf",dis + i,cost + i);
	cost[++n] = 0,dis[n] = totaldis;
	dfs(0,0,0);
	if(res < 1e18) printf("%.2f",res);
	else puts("No Solution");
	return 0;
}
```

### P2114 [NOI2014]起床困难综合症

拆位考虑，对于每一位，考虑初始为 $0$ 或为 $1$ 时最终会变成什么，然后随便搞搞就行了。

然后一个最高位的贡献超过所有低位，所以贪心从高往低，选零有贡献就选零，否则选一有贡献就选一。

```cpp
#include <cstdio>
#include <algorithm>
#include <cstring>
const int maxn = 1e5 + 100;
#define int long long
int n,m,opt[maxn],w[maxn];
int zero,one;
signed main()
{
	scanf("%lld %lld",&n,&m);
	for(int i = 1;i<=n;++i)
	{
		static char s[10];
		scanf("%s %lld",s + 1,w + i);
		if(s[1] == 'A') opt[i] = 1;
		else if(s[1] == 'O') opt[i] = 2;
		else opt[i] = 3;
	}
	one = (1ll << 60) - 1;
	for(int i = 1;i<=n;++i) if(opt[i] == 1) zero &= w[i], one &= w[i]; else if(opt[i] == 2) zero |= w[i], one |= w[i];
	else zero ^= w[i], one ^= w[i];
	int res = 0, now = 0;
	for(int i = 40;i >= 0;--i) 
	{
		if(zero & (1ll << i)) res += 1ll << i;
		else if(now + (1ll << i) <= m && (one & (1ll << i))) res += 1ll << i,now += 1ll << i;
	}
	printf("%lld\n",res);
	return 0;
}
```

### P2827 蚯蚓

跟今年的贪吃蛇一样的双队列优化。

首先全局加看上去就很假，转化为单点减即可，然后容易想到用 multiset 之类的东西来维护一下最大值，就可以做到带 $\log$ 的解法。

正解要求 $O(m)$，需要发现题目隐藏的单调性。

一个自然的想法是按照被切过的蚯蚓和没切过的进行分类。

假定一开始切了一条长度为 $a$ 的蚯蚓，后来 $t$ 秒后切了一条当时长度为 $b$ 的蚯蚓。

在切完 $b$ 时，其长度分别为 $\lfloor pa \rfloor + t \times q$，$a - \lfloor pa \rfloor + t \times q$，$\lfloor p \times (b + (t-1) \times q)\rfloor$，$b + (t-1) \times q - \lfloor p \times (b + (t-1) \times q)\rfloor$

一开始有 $a>b$：

$$
\lfloor pa \rfloor + t \times q - \lfloor p \times (b + (t-1) \times q)\rfloor
$$

$$
\implies \lfloor p \times (a + \dfrac{1}{p} \times t \times q) \rfloor - \lfloor p \times (b + (t-1) \times q)\rfloor
$$

$$
\implies \lfloor p \times (a + \dfrac{1}{p} \times t \times q -  b - (t-1) \times q)\rfloor
$$

$$
\implies \lfloor p \times (a - b + ((\dfrac{1}{p} - 1) \times t + 1)\times q)\rfloor
$$

这东西显然是正的，那么先分的左半边会大于后分的左半边。

另一边也是差不多的证明方法。

那么乱搞一下，分成三类：没切过的，切了的左边的，切了的右边的，就可以线性了。
一开始给所有蚯蚓排序，总时间复杂度 $O(n \log n + m)$.

然后数组注意要开很多倍大小，因为要考虑到出队的元素空间依然占用，需要开操作数的两倍。
最后把三个队列归并起来，炫酷拉风的三路归并科技。

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
const int maxm = 1e7 + 100;
int n, m, u, v, q, t, a[maxn];
int q1[maxn], q2[maxm*2], q3[maxm*2], qh1, qt1, qh2, qt2, qh3, qt3;
int Q[maxm*3], qt, qh;
inline int cut(int x, int k)
{
    int w = x + q * (k - 1);
    int l = 1ll * w * u / v, r = w - l;
    q2[++qt2] = l - q * k, q3[++qt3] = r - q * k;
    return w;
}
int main()
{
    read(n), read(m), read(q), read(u), read(v), read(t);
	for (int i = 1; i <= n; ++i) read(a[i]);
	std::sort(a + 1,a + 1 + n);
    qh1 = qh2 = qh3 = 1;
    for (int i = n; i; --i) q1[++qt1] = a[i];
	for (int i = 1; i <= m; ++i) 
	{
        int maxx = -(1 << 30), res = 0;
        if (qt1 >= qh1) maxx = std::max(maxx, q1[qh1]);
        if (qt2 >= qh2) maxx = std::max(maxx, q2[qh2]);
        if (qt3 >= qh3) maxx = std::max(maxx, q3[qh3]);
        if (qt1 >= qh1 && maxx == q1[qh1]) res = cut(q1[qh1++], i);
        else if (qt2 >= qh2 && maxx == q2[qh2]) res = cut(q2[qh2++], i);
        else if (qt3 >= qh3 && maxx == q3[qh3]) res = cut(q3[qh3++], i);
        if (i % t == 0) printf("%d ", res);
    }
	puts("");
	while(qt1 >= qh1 || qt2 >= qh2 || qt3 >= qh3)
	{
       int maxx = -(1 << 30);
       if (qt1 >= qh1) maxx = std::max(maxx, q1[qh1]);
       if (qt2 >= qh2) maxx = std::max(maxx, q2[qh2]);
       if (qt3 >= qh3) maxx = std::max(maxx, q3[qh3]);
       if (qt1 >= qh1 && maxx == q1[qh1]) Q[++qt] = q1[qh1++];
       else if (qt2 >= qh2 && maxx == q2[qh2]) Q[++qt] = q2[qh2++];
       else if (qt3 >= qh3 && maxx == q3[qh3]) Q[++qt] = q3[qh3++];
    }
    for (int i = t; i <= qt; i += t) printf("%d ", Q[i] + m * q);
    return 0;
}
```

### P7078 贪吃蛇

足够聪明的蛇蛇。

首先有一个结论：如果一条蛇吃完不是最弱的，它一定敢吃。

一条蛇吃完后，下一条蛇如果吃，吃完后一定比它弱，而如果下一条蛇吃，就说明下一条蛇吃完不会被吃，那么它比下一条蛇强就更不会被吃。如果下一条蛇不吃，那么游戏结束，它也不会被吃。

效仿蚯蚓，我们来 YY 一下，也许每条吃完的蛇长度都是单调减小的？最弱的越来越大，因为没有蛇吃完会变成最弱，所以最弱单调递增，最强单调递减，最强吃完最弱后也单调递减。

然而此时，你会发现一个问题：最弱不一定单增！~~因为你足够聪明。~~

因为如果有一条蛇足够大胆，它会发现下一条蛇不敢吃它，因为下一条蛇吃了就会被吃，所以它敢吃而变成最弱。

这就是单调性被破坏了，此时它敢不敢吃，完全取决于它的下一条蛇敢不敢吃。

它的下一条蛇敢不敢吃呢？如果它吃完不是最弱的，它一定敢吃，否则又要取决于下一条蛇。

这就是个递归，边界在哪里呢？如果只有两条蛇，那么一定敢吃。

这个阶段也是有单调性的，每条蛇吃完都是最弱，顺次模拟下去。

细节恶心死了，居然逼得我又写起了恶心封装。

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
const int maxn = 3e6 + 100;
int T, n, a[maxn], qh1, qt1, qh2, qt2, eaten;
struct snake
{
    int val, id;
    snake() { val = id = 0; }
    snake(int val, int id) { this->val = val, this->id = id; }
    bool operator<(const snake& b) const { return val != b.val ? val < b.val : id < b.id; }
    bool operator>(const snake& b) const { return val != b.val ? val > b.val : id > b.id; }
} q1[maxn], q2[maxn];
inline snake getmax()
{
    snake res;
    if (qt1 >= qh1) res = std::max(res, q1[qh1]);
    if (qt2 >= qh2) res = std::max(res, q2[qh2]);
    return res;
}
inline snake getmin()
{
    snake res;
    res.val = 1 << 30;
    if (qt1 >= qh1) res = std::min(res, q1[qt1]);
    if (qt2 >= qh2) res = std::min(res, q2[qt2]);
    return res;
}
inline void popmax()
{
    if(qt1 >= qh1 && qt2 >= qh2)
    {
        if (q1[qh1] > q2[qh2]) ++qh1; else ++qh2;
    }
    else if (qt1 >= qh1) ++qh1;
    else ++qh2;
}
int lastmin;
inline void popmin()
{
    if(qt1 >= qh1 && qt2 >= qh2) { if (q1[qt1] < q2[qt2]) --qt1, lastmin = 1; else --qt2, lastmin = 2; }
    else if (qt1 >= qh1) --qt1, lastmin = 1;
    else --qt2, lastmin = 2;
}
inline void recovermin()
{
    if(lastmin == 1) ++qt1; else if(lastmin == 2) ++qt2;
}
inline snake getsecmin()
{
    snake res;
    popmin(), res = getmin(), recovermin();
    return res;
}
inline int totalcount() { return std::max(0, qt1 - qh1 + 1) + std::max(0, qt2 - qh2 + 1); }
bool dfs()//搜索敢不敢吃
{
    if(totalcount() == 2) return 1;
    snake maxx = getmax(), minn = getmin(), secmin = getsecmin();
    maxx.val -= minn.val;
    if(maxx > secmin) return 1;
    else
    {
        popmax(), popmin(), q2[++qt2] = maxx;
        return !dfs();
    }
}
int main()
{
    read(T);
    for (int t = 1; t <= T; ++t)
    {
        if(t == 1) { read(n); for (int i = 1; i <= n; ++i) read(a[i]); }
        else { int k; read(k); for (int i = 1, p, x; i <= k; ++i) read(p), read(x), a[p] = x; }
        qh1 = qh2 = 1, qt1 = qt2 = 0, eaten = 0;
        for (int i = n; i; --i) q1[++qt1] = (snake){a[i], i};
        while(totalcount() >= 2)
        {
            snake maxx = getmax(), minn = getmin(), secmin = getsecmin();
            maxx.val -= minn.val;
            if (maxx > secmin) popmax(), popmin(), q2[++qt2] = maxx, ++eaten;
            else { eaten += dfs(); break; }
        }
        printf("%d\n", n - eaten);
    }
    return 0;
}
```

### P2119 魔法阵

可以考虑先按 $X$ 进行排序，然后依次数数。
又是神奇的数数题啊。

第一种情况，$X_a < X_b < X_c < X_d$，此时 $a = i$.

那么就是要统计 $a < b < c < d$ 中，满足 $X_a = X_b  + 2 \times X_c - 2 \times X_d$，且 $X_a > X_b - \dfrac{(X_c - X_b)}{3}$ 的数量。

值域比较小，感觉要从值域上做手脚了。

$X_a > \dfrac{4}{3} \times X_b - X_c$

首先，可以发现，$X_b - X_a = 2 \times (X_d - X_c)$ 这个式子是有意义的，可以考虑为两个值的差两倍于令两个值。并且可以发现，$X_b - X_a$ 是偶数时才合法。

然后其实 $X_b - X_a < \dfrac{X_c - X_b}{3}$ 也是有意义的，限制了 $c$ 点的范围不能太远，再结合值域，感觉就是打个暴力了。

之前的式子应该都全部白化简了。

可以考虑枚举 $X_b$，然后再枚举 $X_a$，保证 $X_b - X_a$ 是偶数，然后 $X_b - X_a = 2 \times (X_d - X_c)$，因此上界是 $10000$，而每次跳两步，其实是 $O(5000^2)$ 的。

此时只需要查询对于所有合法的 $X_c$，差分数组打个标记，然后对每个 $X_c$ 的对应的 $X_d$ 也标记一下，这时候发现一开始枚举差值会比较好。

乱搞一下就行了，反正是普及组的题目。

大概就是枚举差值，然后做个可行对数前缀和，然后再差分做区间修改。细节多，多调调就行了。普及组的假题。

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
int n, m, a[maxn], num[maxn];
int A[maxn], B[maxn], C[maxn], D[maxn];
int tmp[maxn];
int pairnum[maxn];
signed main()
{
    read(n), read(m);
    int maxx = 0, minn = 200000;
    for (int i = 1; i <= m; ++i) num[read(a[i])]++, maxx = std::max(maxx, a[i]), minn = std::min(minn, a[i]);
    for (int dis = 1;dis < maxx;++dis)
    {
        int up = maxx - dis - 6 * dis - 1;
        if(6 * dis < maxx - dis - up - 1) ++up;
        if(up < 1) break;
        int minb = 1 + 2 * dis;
//        printf("dis:%d up %d minb %d\n",dis,up,minb);
        for (int i = 0; i <= up + 1; ++i) tmp[i] = 0;
        for (int b = up; b >= minb; --b)
        {
            int a = b - 2 * dis;
            if(a < 1) break;
            pairnum[b] = num[b] * num[a];
        }
        for (int i = minb + 1; i <= up; ++i) pairnum[i] += pairnum[i - 1];
        for (int c = maxx - dis;c > 1; --c)
        {
            int d = c + dis;
            if(d < 1) break;
            if(!num[c] || !num[d]) continue;
            int b = c - 6 * dis - 1;
            if(b < 1) break;
            if(6 * dis < c - b - 1) ++b;
            int a = b - 2 * dis;
            if (a < 1) break;
            tmp[b] += num[c] * num[d], C[c] += num[d] * pairnum[b], D[d] += num[c] * pairnum[b];
        }
        for (int i = up - 1; i >= minb; --i) tmp[i] += tmp[i + 1];
        for (int b = up; b >= minb; --b)
        {
            int a = b - 2 * dis;
            if(a < 1) break;
            if(!num[a] || !num[b]) continue;
            A[a] += num[b] * tmp[b], B[b] += num[a] * tmp[b];
        }
    }
    for (int i = 1; i <= m; ++i) printf("%d %d %d %d\n",A[a[i]],B[a[i]],C[a[i]],D[a[i]]);
    return 0;
}
```

### P2822 组合数问题

小常数的 $O(n^2k)$ 应该是能过的，只要把取模避免掉就可以了，那么直接统计阶乘中对应指数位的数值即可。
做个二位前缀和之类的东西。
空间不够用，那就质因数分解，只有 $2$，$3$，$5$，$7$，$11$，$13$，$17$，$19$ 这么一共 $8$ 个质数，那么空间就够了。

然而答案记录还是要开满，然后一眼突然发现……原来 $k$ 是固定的？我在干啥？

sb 题，直接递推组合数取模即可。

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
const int maxn = 2100;
int t,k;
int C[maxn][maxn],sum[maxn][maxn];
int main()
{
    read(t), read(k);
    C[0][0] = 1;
    for (int i = 1; i <= 2000; ++i)
    {
        C[i][0] = 1;
        for (int j = 1; j <= i; ++j)
            C[i][j] = (C[i - 1][j] + C[i - 1][j - 1]) % k, sum[i][j] = (C[i][j] == 0);
    }
    for (int i = 1; i <= 2000; ++i) for (int j = 1; j <= 2000; ++j) sum[i][j] += sum[i - 1][j];
    for (int i = 1; i <= 2000; ++i) for (int j = 1; j <= 2000; ++j) sum[i][j] += sum[i][j - 1];
    for (int i = 1, n, m; i <= t; ++i) read(n), read(m), printf("%d\n", sum[n][std::min(n, m)]);
    return 0;
}
```

### LOJ 6520. 「ICPC PacNW 2017 Div.1」David 的旅程

突然插入这道题，是为了给回家路线做个小铺垫。

考虑从航班来动态规划，有一个很 naive 的状态设置：设 $f(i,j)$ 为当前时间为 $i$，目前在城市 $j$ 的最小挫败感。

对于每一条航班，都有：

$$
f(e_i,v_i) = \min \limits _{0 \le k \le s_i}f(k,u_i) + (s_i-k)^2
$$

这种形式看上去就很斜率优化，然而这二维的状态实在是优化不起啊。
考虑想办法拆掉一维，既然时间已经在转移方程中出现了，似乎只能想办法拆掉城市了。

每条路径，更新的都是 $v_i$ 城市中的答案，并且只会更新一个点。而对于路径 $u_i \to v_i$，$v_i$ 城市的状态需要从 $u_i$ 城市转移。

这可以看做一个单点更新操作与一个查询操作，把它们拆开考虑。

对于一个路径 $u \to v$，要求 $v_i \le u$ 的所有修改操作都被加入，就能进行转移。然而稍一思考，就会发现，这东西可以成环，没法做啊。

滚回来想办法拆时间，可以发现对于一条路径，它需要考虑的路径只有到达时间小于等于它的出发时间的路径。

因此对路径的出发时间进行排序，然后用双指针的技巧动态插入 $f(t,v_i)$ 这样的修改操作，然后查询一次，又把查询造成的修改扔到待应用队列中。

对每个站点，都可以大力维护一个单调队列，插入会满足时间单调递增，也就是横坐标单调递增，所以这是很自然的。

然后把式子拆开吧：

$$
f(e_i,v_i) = f(k,u_i) + s_i^2 + k^2 - 2 \times s_i \times k
$$

整理一下：

$$
2 \times s_i \times k - s_i^2 + f(e_i,v_i) = f(k,u_i) + k^2
$$

斜率是 $2 \times s_i$，截距是 $f(e_i,v_i) - s_i^2$，横坐标是 $k$，纵坐标是 $f(k,u_i) + k^2$，要求最小化截距。

```cpp
#include <cstdio>
#include <algorithm>
#include <queue>
#include <ctype.h>
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
#define int long long
const int maxn = 2e5 + 100;
const int maxm = 1e6 + 100;
int n, m;
struct path { int u, v, s, t; } A[maxn];
struct point { int x; long long y; };
vector<point> V[maxn];
vector<pair<int, point> > G[maxm];
int head[maxn];  //queue head
inline bool pcmp(const point& a, const point& b, const point& c, const point& d) { return 1.0 * (a.y - b.y) * (c.x - d.x) <= 1.0 * (c.y - d.y) * (a.x - b.x); }
inline bool pcmp(const point &a,const point &b,const int k) { return (a.y - b.y) < 1.0 * k * (a.x - b.x); }
inline bool cmp(const path& a, const path& b) { return a.s < b.s; }
inline void ins(int x, point p)
{
    int ed;
    while ((ed = V[x].size() - 1) > head[x] && !pcmp(V[x][ed - 1], V[x][ed], V[x][ed], p)) V[x].erase(--V[x].end());
    V[x].push_back(p);
}
inline point pop(int x, int k) { while ((int)(V[x].size()) > head[x] + 1 && !pcmp(V[x][head[x]], V[x][head[x] + 1], k)) ++head[x]; return V[x][head[x]]; }
signed main()
{
    read(n), read(m);
    for (int i = 1; i <= m; ++i) read(A[i].u), read(A[i].v), read(A[i].s), read(A[i].t);
    sort(A + 1, A + 1 + m, cmp);
    for (int i = 2; i <= n; ++i) V[i].push_back((point){0, (1ll << 60)});
    V[1].push_back((point){0, 0});
    int p = 0;
    long long ans = 1ll << 60;
    for (int i = 1; i <= m; ++i)
    {
        while (p < A[i].s) for (vector<pair<int, point> >::iterator it = G[++p].begin(); it != G[p].end(); ++it) ins(it->first, it->second);
        point res = pop(A[i].u, 2ll * A[i].s);
        long long f = res.y - 2ll * A[i].s * res.x + 1ll * A[i].s * A[i].s;
        if (A[i].v == n) ans = std::min(ans, f);
        G[A[i].t].push_back(make_pair(A[i].v, (point){A[i].t, f + 1ll * A[i].t * A[i].t}));
    }
    printf("%lld\n", ans);
    return 0;
}
```

### P5468 [NOI2019]回家路线

似乎是差不多的做法了，只是贡献统计变了罢了。

$$
f(v,s_i) = f(u,k) + A(s_i - k)^2 + B(s_i - k) + C
$$

$$
\implies f(v,s_i) = f(u,k) + A \times s_i^2 + A \times k^2 - 2 \times A \times s_i \times k + B \times s_i - B \times k + C
$$

$$
\implies 2 \times A \times s_i \times k + f(v,s_i) - A \times s_i^2 - B \times s_i - C = f(u,k) + A \times k^2 - B \times k
$$

观察一下，斜率是 $2 \times A \times s_i$，截距是 $f(v,s_i) - A \times s_i^2 - B \times s_i - C$，横坐标是 $k$，纵坐标是 $f(u,k) + A \times k^2 - B \times k$.

求 $f(v,s_i) = y - k \times x + A \times s_i^2 + B \times s_i + C$

要求最小化截距，可以维护一个斜率单调递增的下凸包。

```cpp
#include <cstdio>
#include <algorithm>
#include <vector>
#include <ctype.h>
using namespace std;
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
const int maxn = 1e5 + 100;
const int maxm = 1e6 + 100;
const int maxt = 4e4 + 100;
int n, m, A, B, C;
struct path { int u, v, s, t; } P[maxm];
inline bool cmp(const path& a, const path& b) { return a.s < b.s; }
struct point { int x; long long y; };
vector<point> S[maxn];
int head[maxn];
vector<pair<int,point> > M[maxt];
inline bool fcmp(const point& a, const point& b, const point& c, const point& d)
{
    return (a.y - b.y) * (c.x - d.x) < (c.y - d.y) * (a.x - d.x);
}
inline bool kcmp(const point &a,const point &b,long long k) { return a.y - b.y > k * (a.x - b.x); }
inline void ins(int pos, point p)
{
    for (int ed = (int)(S[pos].size()) - 1; ed - 1 >= head[pos] && !fcmp(S[pos][ed - 1], S[pos][ed], S[pos][ed], p); --ed)
        S[pos].pop_back();
    S[pos].push_back(p);
}
inline point pop(int pos, long long k)
{
    while (head[pos] + 1 < (int)(S[pos].size()) && kcmp(S[pos][head[pos]],S[pos][head[pos] + 1],k)) ++head[pos];
    return S[pos][head[pos]];
}
int main()
{
    read(n), read(m), read(A), read(B), read(C);
    for (int i = 1; i <= m; ++i) read(P[i].u),read(P[i].v),read(P[i].s),read(P[i].t);
    sort(P + 1, P + 1 + m, cmp);
    int p = 0;
    long long ans = 1ll << 60;
    for (int i = 2; i <= n; ++i) S[i].push_back((point){0, (1ll << 50)});
    S[1].push_back((point){0, 0ll});
    for (int i = 1; i <= m; ++i) 
    {
        while (p < P[i].s)
            for (vector<pair<int, point> >::iterator it = M[++p].begin(); it != M[p].end(); ++it) ins(it->first, it->second);
        point res = pop(P[i].u, 2.0 * A * P[i].s);
        long long f = res.y - 2ll * A * P[i].s * res.x + 1ll * A * P[i].s * P[i].s + 1ll * B * P[i].s + C;
        if (P[i].v == n) ans = min(ans, f + P[i].t);
        else M[P[i].t].push_back(make_pair(P[i].v, (point){P[i].t, f + 1ll * A * P[i].t * P[i].t - 1ll * B * P[i].t}));
    }
    printf("%lld\n", ans);
    return 0;
}
```

### P3960 列队

可以发现，每次离队一个人，只会改变一行一列一点。

变化量比较小，所以可能不是找规律。

考虑一个点 $(x,y)$，它目前的编号是什么。这取决于它被如何移动，具体地，它左方的点被删去，它就会被移动。除此之外，第 $m$ 列是很特殊的一列，每次删点都会导致其移动。

考虑一下，用动态开点线段树维护，具体看代码吧。

大概就是维护删了多少个点，然后在树上二分第 $k$ 个点这样的东西，在结尾插入又得重写个判断的鬼东西。这种毕竟还是平衡树做比较正统。

```cpp
#include <cstdio>
#include <algorithm>
#include <set>
#include <cctype>
using namespace std;
const int bufSize = 1e6;
inline char nc()
{
	#ifdef DEBUG
	return getchar();
	#endif
	static char buf[bufSize],*p1 = buf,*p2 = buf;
	return p1==p2&&(p2=(p1=buf)+fread(buf,1,bufSize,stdin),p1==p2)?EOF:*p1++;
}
template<typename T>
inline T read(T &r)
{
	static char c;
	r = 0;
	for(c = nc(); !isdigit(c); c = nc());
	for(; isdigit(c); c = nc()) r = r * 10 + c - 48;
	return r;
}
const int maxn = 3e5 + 100;
int n, m, q;
int cnt[maxn],root[maxn], sum[maxn * 20], L[maxn * 20], R[maxn * 20], ind;
long long val[maxn * 20];
inline int get(int l,int r,int p)
{
	if(p == n + 1) return r <= n ? r - l + 1 : (l <= n ? n - l + 1 : 0);
	return r < m ? r - l + 1 : (l < m ? m - l : 0);
}
long long ask(int askroot,int l,int r,int &p, int pos)
{
	if(!p) p = ++ind;
	sum[p]++;
	if(l == r) return val[p] ? val[p] : val[p] = (askroot == n + 1) ? 1ll * l * m : 1ll * m * (askroot-1) + l;
	int mid = l + r >> 1,lsum = get(l,mid,askroot) - sum[L[p]];
	return pos <= lsum ? ask(askroot,l,mid,L[p],pos) : ask(askroot,mid + 1,r,R[p],pos - lsum);
}
void modify(int askroot,int l,int r,int &p, int pos, long long k)
{
	if(!p) p = ++ind;
	--sum[p];
	if(l == r) return (void)(val[p] = k);
	int mid = l + r >> 1;
	if(pos <= mid) modify(askroot,l,mid,L[p],pos,k); else modify(askroot,mid + 1,r,R[p],pos,k);
}
int main()
{
	read(n), read(m), read(q);
	int maxx = std::max(n,m) + q + 10;
	for(int i = 1,x,y;i<=q;++i) 
	{
		static long long res;
		read(x),read(y);
		if(y == m) 
		{
			res = ask(n + 1, 1, maxx, root[n + 1], x);
			printf("%lld\n",res);
			modify(n + 1, 1, maxx, root[n + 1], n + (++cnt[n + 1]), res);
		}
		else 
		{
			res = ask(x, 1, maxx, root[x], y);
			printf("%lld\n", res);
			modify(n + 1, 1, maxx, root[n + 1], n + (++cnt[n + 1]), res);
			res = ask(n + 1,1,maxx,root[n + 1],x);
			modify(x, 1, maxx,root[x],m - 1 + (++cnt[x]), res);
		}
	}

	return 0;
}
```

### P1084 疫情控制

树上最大值，考虑二分答案。

转化为检验问题，可以发现一个点往上一定更优，所以无脑让所有点往上跳。

每个支配点都往上跳，在跳到的点把支配点存下来，然后大力搜索一遍检查首都的每个子树是否满足。

对于不满足的子树，必须从别的子树跳过来，可以发现必须是别的子树的根节点才有可能，维护一个 multiset 用于乱跳即可。还有一个点：如果某个跳到 $1$ 的点，剩余的不足以跳回去，那么一定让它不跳，留着支配它的源子树。

```cpp
#include <cstdio>
#include <algorithm>
#include <set>
#include <cctype>
using namespace std;
const int bufSize = 1e6;
inline char nc()
{
	#ifdef DEBUG
	return getchar();
	#endif
	static char buf[bufSize],*p1 = buf,*p2 = buf;
	return p1==p2&&(p2=(p1=buf)+fread(buf,1,bufSize,stdin),p1==p2)?EOF:*p1++;
}
template<typename T>
inline T read(T &r)
{
	static char c;
	r = 0;
	for(c = nc(); !isdigit(c); c = nc());
	for(; isdigit(c); c = nc()) r = r * 10 + c - 48;
	return r;
}
const int maxn = 3e5 + 100;
struct node
{
	int to,next,val;
}E[maxn << 1];
int head[maxn],tot;
inline void add(int x,int y,int val){E[++tot].next = head[x],head[x] = tot,E[tot].to = y,E[tot].val = val;}
int n, m, fa[maxn][19], S[maxn], bel[maxn], tag[maxn], isleaf[maxn], can[maxn], more[maxn];
long long dis[maxn][19], wdis;
void dfs(int u)
{
	isleaf[u] = 1;
	for(int i = 1; i < 19; ++i) fa[u][i] = fa[fa[u][i-1]][i-1], dis[u][i] = dis[u][i-1] + dis[fa[u][i-1]][i-1];
	for(int p = head[u]; p; p = E[p].next)
	{
		int v = E[p].to;
		if(v == fa[u][0]) continue;
		bel[v] = bel[u], fa[v][0] = u, dis[v][0] = E[p].val, dfs(v),isleaf[u] = 0;
	}
}
void dfs1(int u)
{
	if(tag[u]) return;
	if(isleaf[u]) return (void)(can[bel[u]] = 0);
	for(int p = head[u];p;p=E[p].next)
	{
		int v = E[p].to;
		if(v == fa[u][0]) continue;
		dfs1(v);
	}
}
multiset<long long> s[maxn];
bool check()
{
	//printf("\nbegin:%lld\n",wdis);
	for(int i = 1;i <= n;++i) more[i] = tag[i] = can[i] = 0,s[i].clear();
	multiset<long long> A;
	for (int i = 1; i <= m; ++i) 
	{
		long long left = wdis;
		int u = S[i], belu = bel[u];
		for(int j = 18; u != 1 && j >= 0; --j) 
			if(left - dis[u][j] >= 0) 
				left -= dis[u][j], u = fa[u][j];
		if(u == 1) s[belu].insert(left), more[belu] = 1,A.insert(left);//, printf("top:%d %lld\n",belu,left);
		else tag[u] |= 1;
	}

	for(int p = head[1];p;p=E[p].next)
	{
		int v = E[p].to;
		can[v] = 1;
		if(more[v])
		{
			dfs1(v);
			if(!can[v] && *s[v].begin() <= dis[v][0]) can[v] = 1, A.erase(A.find(*s[v].begin()));
		
		}
		else dfs1(v);
	}
	for(int p = head[1];p;p=E[p].next)
	{
		int v = E[p].to, d = dis[v][0];
		if(!can[v]) 
		{
			set<long long>::iterator it = A.lower_bound(d);
			if(it == A.end()) return 0;
			//printf("need: %d %lld\n",v,*it);
			A.erase(it);
		}
	}
	return 1;
}
int main()
{
	read(n);
	for(int i = 1, u, v, w; i < n; ++i) read(u), read(v), read(w), add(u, v, w), add(v, u, w);
	read(m);
	for(int i = 1; i <= m; ++i) read(S[i]);
	for(int i = 0;i < 19;++i) fa[1][i] = 1,dis[1][i] = 0;
	for(int p = head[1];p;p=E[p].next) 
	{
		int v = E[p].to;
		bel[v] = v, fa[v][0] = 1, dis[v][0] = E[p].val, dfs(v);
	}
	long long maxdis = 0;
	for(int i = 1; i <= n; ++i) maxdis = std::max(maxdis, dis[i][23]);
	long long l = 0, r = maxdis << 1,ans = -1;
	while(l <= r)
	{
		wdis = (l + r) >> 1;
		if(check()) r = wdis - 1,ans = wdis;
		else l = wdis + 1;
	}
	printf("%lld\n",ans);
	return 0;
}
```

### P1081 开车旅行

首先确定每个点移动后到达哪个点，绝对值不好处理，可以用 set 二分判断。

然后倍增一下，处理每个点当前点为 $A,B$ 时跳 $2^i$ 步之后会到哪个点，顺便处理距离。

然后就可以在某个状态下一步到位。

细节比较恶心。代码有很多优化空间。

```cpp
#include <cstdio>
#include <set>
#include <algorithm>
#include <cctype>
using namespace std;
const int bufSize = 1e6;
#define DEBUG
inline char nc()
{
	#ifdef DEBUG
	return getchar();
	#endif
	static char buf[bufSize],*p1,*p2;
	return p1==p2&&(p2=(p1=buf)+fread(buf,1,bufSize,stdin),p1==p2)?EOF:*p1++;
}
template<typename T>
inline T read(T &r)
{
	static char c;
	static int flag;
	r = 0,flag = 1;
	for(c = nc(); !isdigit(c); c = nc()) if(c == '-') flag = -1;
	for(;isdigit(c);c=nc()) r = r * 10 + c - 48;
	return r *= flag;
}
const int maxn = 1e5 + 100;
const int maxk = 18;
int n, m, h[maxn];
struct node
{
	int to;
	long long dis,adis,bdis;
}f[maxn][maxk][2];
template<typename T> inline T myabs(const T &x){return x > 0 ? x : -x;}
inline void ins(int i,pair<int,int> p)
{
	int d = myabs(h[i] - p.first),id = p.second;
	if(!f[i][0][1].to) f[i][0][1] = (node){id,d,0,d};
	else
	{
		if(f[i][0][1].dis > d || (f[i][0][1].dis == d && h[f[i][0][1].to] > h[id])) 
		{
			f[i][0][0] = (node){f[i][0][1].to,f[i][0][1].dis,f[i][0][1].dis,0};
			f[i][0][1] = (node){id,d,0,d};
		}
		else if(!f[i][0][0].to || (f[i][0][0].dis > d || (f[i][0][0].dis == d && h[f[i][0][0].to] > h[id]))) 
			f[i][0][0] = (node){id,d,d,0};
	}
}
int main()
{
	read(n);
	for(int i = 1;i <= n;++i) read(h[i]);
	set<pair<int,int> > S;
	for(int i = n;i;--i)
	{
		set<pair<int,int> >::iterator it = S.lower_bound(make_pair(h[i],0));
		if(it != S.end()) ins(i,*it);
		if(it != S.begin()) 
		{
			ins(i,*--it);
			if(it != S.begin()) ins(i,*--it),++it;
			++it;
		}
		if(it == S.end()) {S.insert(make_pair(h[i],i));continue;}
		++it;
		if(it != S.end()) ins(i,*it);
		else {S.insert(make_pair(h[i],i));continue;}
		++it;
		if(it != S.end()) ins(i,*it);
		else {S.insert(make_pair(h[i],i));continue;}
		S.insert(make_pair(h[i],i));
	}
	for(int j = 1;j<18;++j) 
		for(int i = 1; i <= n; ++i) 
		{
			node t1 = f[i][j-1][0];
			node t2 = f[t1.to][j-1][j == 1];
			//if(!t2.to) break;
			f[i][j][0] = (node){t2.to,t1.dis + t2.dis,t1.adis + t2.adis,t1.bdis + t2.bdis};
			t1 = f[i][j-1][1];
			t2 = f[t1.to][j-1][j != 1];
			f[i][j][1] = (node){t2.to,t1.dis + t2.dis,t1.adis + t2.adis,t1.bdis + t2.bdis};
		}
	int x0,anss = 0;
	read(x0);
	long long ansadis = 0,ansbdis = 0;
	for(int i = 1;i <= n;++i) 
	{
		int now = i,status = 0,left = x0,adis = 0,bdis = 0;
		for(int j = 17;j>=0;--j) 
			if(f[now][j][status].to && left >= f[now][j][status].dis) 
			{
				adis += f[now][j][status].adis,bdis += f[now][j][status].bdis;
				left -= f[now][j][status].dis,now = f[now][j][status].to;
			}
		if(!anss) ansadis = adis,ansbdis = bdis,anss = i;
		else if(bdis == 0) 
		{
			if(ansbdis == 0 && h[i] > h[anss]) anss = i,ansadis = adis,ansbdis = bdis;
		}
		else if((adis * ansbdis < bdis * ansadis) || 
			(adis * ansbdis == bdis * ansadis && h[i] > h[anss])) 
				ansadis = adis,ansbdis = bdis,anss = i;
	}
	printf("%d\n",anss);
	read(m);
	while(m--)
	{
		static int now,x;
		read(now),read(x);
		long long adis = 0,bdis = 0;
		for(int j = 17;j>=0;--j) 
			if(f[now][j][0].to && x >= f[now][j][0].dis)
				adis += f[now][j][0].adis,bdis += f[now][j][0].bdis,
				x -= f[now][j][0].dis,now = f[now][j][0].to;
		printf("%lld %lld\n",adis,bdis);
	}
	return 0;
}
```

### P5688 [CSP-SJX2019]散步

毒瘤题？

考虑每个人都会前往最近的出口，花费一定的时间。只需要每次处理最快到达的人，就不会有什么问题。

如何处理？可以发现，如果最快的人离开后，出口仍有容量，则一切不变。如果容量为零，那么该出口相当于被删除。

被删除后，原先最近出口是该出口的人，对应的值都会变化。一个 naive 的想法就是对每个出口储存对应的人的信息，然后暴力修改。

然而这样，一个人的信息被多次修改，复杂度是不可接受的。

可以发现，在位置上，这些最近出口相同的人是连续分布的，可以考虑求出这个区间，然后用数据结构区间修改其花费的时间。

我选择使用线段树维护 $t$ 与 $id$，即每个人前往最近的出口花费的最小时间与对应的最近出口，顺时针逆时针分开建树。

可以用一个链表来维护动态删除出口。

需要讨论绕过原点的情况，恶心，十分恶心，超级恶心。

```
#include <algorithm>
#include <cstdio>
#include <ctype.h>
const int bufSize = 1e6;
using namespace std;
//#define DEBUG
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
int n, m, L;
int pre[maxn], nex[maxn], pos[maxn], limit[maxn];
struct node
{
    int pos, id, direction, near, neardis;
} A[maxn];
inline bool cmp(const node& a, const node& b) { return a.pos < b.pos; }
const int inf = 1 << 30;
struct segtree
{
    int minn[maxn << 2], minid[maxn << 2], exitid[maxn << 2], addtag[maxn << 2], exittag[maxn << 2];
#define ls p << 1
#define rs p << 1 | 1
    inline void pushup(int p)
    {
        if (minn[ls] < minn[rs] || (minn[ls] == minn[rs] && A[minid[ls]].id < A[minid[rs]].id))
            minn[p] = minn[ls], minid[p] = minid[ls], exitid[p] = exitid[ls];
        else minn[p] = minn[rs], minid[p] = minid[rs], exitid[p] = exitid[rs];
    }
    inline void pushdown(int p)
    {
        if (!addtag[p]) return;
        minn[ls] += addtag[p], minn[rs] += addtag[p], addtag[ls] += addtag[p], addtag[rs] += addtag[p], addtag[p] = 0;
        exitid[ls] = exittag[p], exitid[rs] = exittag[p], exittag[ls] = exittag[p], exittag[rs] = exittag[p], exittag[p] = 0;
    }
    void modify(int l, int r, int p, int ll, int rr, int k, int exit)
    {
        if (l >= ll && r <= rr)
            return (void)(minn[p] += k, addtag[p] += k, exitid[p] = exit, exittag[p] = exit);
        int mid = (l + r) >> 1;
        pushdown(p);
        if (ll <= mid) modify(l, mid, ls, ll, rr, k, exit);
        if (rr > mid) modify(mid + 1, r, rs, ll, rr, k, exit);
        pushup(p);
    }
    void update(int l, int r, int p, int pos, int k)
    {
        if (l == r) return (void)(minn[p] = k);
        int mid = (l + r) >> 1;
        pushdown(p);
        if (pos <= mid) update(l, mid, ls, pos, k); else update(mid + 1, r, rs, pos, k);
        pushup(p);
    }
    void build(int l, int r, int p, int direction)
    {
        if (l == r)
        {
            minid[p] = l, exitid[p] = A[l].near;
            if (A[l].direction == direction) minn[p] = A[l].neardis; else minn[p] = inf;
            return;
        }
        int mid = (l + r) >> 1;
        build(l, mid, ls, direction), build(mid + 1, r, rs, direction), pushup(p);
    }
    void output(int l, int r, int p)
    {
        if (l == r)
        {
            printf("output:id:%d pos:%d near:%d neardis:%d\n", A[l].id, A[l].pos, exitid[p], minn[p]);
            return;
        }
        int mid = l + r >> 1;
        pushdown(p);
        output(l, mid, ls), output(mid + 1, r, rs);
    }
} S, N;
int lower(int x)
{
    int l = 1, r = n, mid, ans = 0;
    while (l <= r)
    {
        mid = (l + r) >> 1;
        if (A[mid].pos <= x) ans = mid, l = mid + 1; else r = mid - 1;
    }
    return ans;
}
int upper(int x)
{
    int l = 1, r = n, mid, ans = n + 10;
    while (l <= r)
    {
        mid = (l + r) >> 1;
        if (A[mid].pos >= x) ans = mid, r = mid - 1; else l = mid + 1;
    }
    return ans;
}
inline void del(int x)
{
    int a = pre[x], b = nex[x];
    pre[b] = a, nex[a] = b;
}
inline void delnode(int x)
{
    if (--limit[x] == 0)
    {
#ifdef DEBUG
        printf("expired:id:%d pos:%d\n", x, pos[x]);
#endif
        if (pos[pre[x]] < pos[x])
        {
            int l = upper(pos[pre[x]] + 1), r = lower(pos[x]);
            if (l <= r)
            {
                if (pos[nex[x]] > pos[x]) N.modify(1, n, 1, l, r, pos[nex[x]] - pos[x], nex[x]);
                else N.modify(1, n, 1, l, r, pos[nex[x]] + L - pos[x], nex[x]);
            }
        }
        else
        {
            int l = lower(pos[x]), r = upper(pos[pre[x]] + 1);
            if (pos[nex[x]] > pos[x])
            {
                if (l > 0 && l <= n) N.modify(1, n, 1, 1, l, pos[nex[x]] - pos[x], nex[x]);
                if (r > 0 && r <= n) N.modify(1, n, 1, r, n, pos[nex[x]] - pos[x], nex[x]);
            }
            else
            {
                if (l > 0 && l <= n) N.modify(1, n, 1, 1, l, pos[nex[x]] + L - pos[x], nex[x]);
                if (r > 0 && r <= n) N.modify(1, n, 1, r, n, pos[nex[x]] + L - pos[x], nex[x]);
            }
        }
        if (pos[nex[x]] > pos[x])
        {
            int l = upper(pos[x]), r = lower(pos[nex[x]] - 1);
            if (l <= r)
            {
                if (pos[pre[x]] < pos[x]) S.modify(1, n, 1, l, r, pos[x] - pos[pre[x]], pre[x]);
                else S.modify(1, n, 1, l, r, pos[x] + L - pos[pre[x]], pre[x]);
            }
        }
        else
        {
            int l = lower(pos[nex[x]] - 1), r = upper(pos[x]);
            if (pos[pre[x]] < pos[x])
            {
                if (l > 0 && l <= n) S.modify(1, n, 1, 1, l, pos[x] - pos[pre[x]], pre[x]);
                if (r > 0 && r <= n) S.modify(1, n, 1, r, n, pos[x] - pos[pre[x]], pre[x]);
            }
            else
            {
                if (l > 0 && l <= n) S.modify(1, n, 1, 1, l, pos[x] + L - pos[pre[x]], pre[x]);
                if (r > 0 && r <= n) S.modify(1, n, 1, r, n, pos[x] + L - pos[pre[x]], pre[x]);
            }
        }
        del(x);
    }
}
#ifdef DEBUG
int ans[maxn];
#endif
inline void alloutput()
{
#ifdef DEBUG
    puts("N:");
    N.output(1, n, 1);
    puts("S:");
    S.output(1, n, 1);
    puts("");
#endif
}
int main()
{
    //freopen("walk.in","r",stdin),freopen("my.out","w",stdout);
    read(n), read(m), read(L);
    for (int i = 1; i <= m; ++i) pre[i] = i - 1, nex[i] = i + 1;
    for (int i = 2; i <= m; ++i) read(pos[i]);
    pre[1] = m, nex[m] = 1, pos[m + 1] = L;
    for (int i = 1; i <= m; ++i) read(limit[i]);
    for (int i = 1; i <= n; ++i) A[i].id = i, read(A[i].direction), read(A[i].pos), A[i].pos %= L;
    std::sort(A + 1, A + 1 + n, cmp);
    int p = 1;
    for (int i = 1; i <= n; ++i)
        if (A[i].direction == 0)
        {
            while (p <= m && A[i].pos > pos[p]) ++p;
            A[i].neardis = pos[p] - A[i].pos, A[i].near = p > m ? p - m : p;
        }
    p = m;
    for (int i = n; i; --i)
        if (A[i].direction == 1)
        {
            while (p > 0 && A[i].pos < pos[p]) --p;
            A[i].neardis = A[i].pos - pos[p], A[i].near = p > m ? 1 : p;
        }
#ifdef DEBUG
    for (int i = 1; i <= n; ++i) printf("id: %d pos: %d near: %d neardis: %d\n", A[i].id, A[i].pos, A[i].near, A[i].neardis);
#endif
    N.build(1, n, 1, 0), S.build(1, n, 1, 1);
#ifdef DEBUG
    alloutput();
#endif
#ifndef DEBUG
    long long ans = 0;
#endif
    for (int t = 1; t <= n; ++t)
    {
        if (N.minn[1] < S.minn[1] || (N.minn[1] == S.minn[1] && A[N.minid[1]].id < A[S.minid[1]].id))
        {
            int exitid = N.exitid[1];
#ifdef DEBUG
            printf("leave: id:%d exitid:%d dis:%d\n", A[N.minid[1]].id, exitid, N.minn[1]);
#endif
            if (limit[exitid] <= 0) continue;
#ifndef DEBUG
            ans ^= 1ll * A[N.minid[1]].id * exitid;
#endif
#ifdef DEBUG
            ans[A[N.minid[1]].id] = exitid;
#endif
            N.update(1, n, 1, N.minid[1], inf);
            delnode(exitid);
        }
        else
        {
            int exitid = S.exitid[1];
#ifdef DEBUG
            printf("leave: id:%d exitid:%d dis:%d\n", A[S.minid[1]].id, exitid, S.minn[1]);
#endif
            if (limit[exitid] <= 0) continue;
#ifndef DEBUG
            ans ^= 1ll * A[S.minid[1]].id * exitid;
#endif
#ifdef DEBUG
            ans[A[S.minid[1]].id] = exitid;
#endif
            S.update(1, n, 1, S.minid[1], inf);
            delnode(exitid);
        }
        alloutput();
    }
#ifndef DEBUG
    printf("%lld\n", ans);
#endif
#ifdef DEBUG
    for (int i = 1; i <= n; ++i) printf("%d %d\n", i, ans[i]);
#endif
    return 0;
}
```