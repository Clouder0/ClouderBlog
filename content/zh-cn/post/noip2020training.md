---  
title: "NOIP2020 前练习笔记"  
date: 2020-11-26T11:10:13+08:00  
slug: "noiptraining2020"  
type: post  
tags:  
  - codeforces  
  - noi  
  - noip  
  - binary-lifting  
  - dyanmic-programming  
  - two-pointers  
  - trie  
  - math  
  - construction  
  - segment-tree  
categories:  
  - Algorithm  
---  
  
  
## P5290 [十二省联考 2019] 春节十二响  
  
题面不是人话，翻译一下。  
  
给一棵树，每个点权值为 $w_i$，需要将节点分组。  
分组要求：每个组内的点，两两不存在祖先——后代关系。  
每个组的代价是组内的最大值，求最小的总代价。  
  
考虑对于 $u$ 相连的所有子树，每棵子树内的点都只能与非该子树内的点在同一段内，也就是说，每棵子树内的组都必须与其他子树的组结合。  
  
由于只与最大值有关，有一个贪心的想法：如果当前子树内某组结合后，不会改变原先的最大值，那么一定结合。  
  
然而需要考虑，当前子树每个组只能与其他子树每个组一一结合，何况还有当前子树内出现最大组，要求其他子树合并的情况，因此需要妥当考虑。  
  
可以发现，每次结合都会减少对应组代价的代价，因此：优先消去代价大的。  
  
两类：当前子树，其他子树，分别取出最大值，比大小，然后小的并入大的中。  
  
注意使用启发式合并，时间复杂度 $O(n\log n)$.  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <queue>  
#include <vector>  
#include <cctype>  
using namespace std;  
const int bufSize = 1e6;  
#define DEBUG  
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
	for(c=nc();!isdigit(c);c=nc());  
	for(;isdigit(c);c=nc()) r=r*10+c-48;  
	return r;  
}  
const int maxn = 2e5 + 100;  
int n,fa[maxn],w[maxn];  
struct node  
{  
	int to,next;  
}E[maxn];  
int head[maxn],tot;  
inline void add(int x,int y){E[++tot].next = head[x],head[x] = tot,E[tot].to = y;}  
priority_queue<int> S[maxn];  
void dfs(int u)  
{  
	vector<int> more;  
	for(int p = head[u];p;p=E[p].next)  
	{  
		int v = E[p].to;  
		dfs(v);  
		if(S[u].size() < S[v].size()) std::swap(S[u],S[v]);  
		while(!S[v].empty())  
			more.push_back(std::max(S[u].top(),S[v].top())),S[u].pop(),S[v].pop();  
		for(vector<int>::iterator it = more.begin();it!=more.end();++it) S[u].push(*it);  
		more.clear();  
	}  
	S[u].push(w[u]);  
	vector<int>().swap(more);  
}  
int main()  
{  
	read(n);  
	for(int i = 1;i<=n;++i) read(w[i]);  
	for(int i = 2;i<=n;++i) read(fa[i]),add(fa[i],i);  
	dfs(1);  
	long long res = 0;  
	while(!S[1].empty()) res += S[1].top(),S[1].pop();  
	printf("%lld\n",res);  
	return 0;  
}  
```  
  
### P4053 [JSOI2007]建筑抢修  
  
一开始想动态规划，然而每个建筑修了没修是不可确定的。  
可以考虑，按报废时间排序。当每个建筑马上报废时进行决策，如果能直接修它，那么修它。  
如果不能直接修它，找到之前修的最耗时的，不修最耗时的而修它，节约时间。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <queue>  
using namespace std;  
const int maxn = 3e5 + 100;  
int n;  
struct node  
{  
	int t1,t2;  
}A[maxn];  
bool cmp(const node &a,const node &b){return a.t2 < b.t2;}  
priority_queue<int> q;  
int main()  
{  
	scanf("%d",&n);  
	for(int i = 1;i<=n;++i) scanf("%d %d",&A[i].t1,&A[i].t2);  
	std::sort(A + 1,A + 1 + n,cmp);  
	int now = 0,res = 0;  
	for(int i = 1;i<=n;++i)  
	{  
		if(now + A[i].t1 <= A[i].t2) now += A[i].t1,q.push(A[i].t1),++res;  
		else if(q.size() && A[i].t1 <= q.top()) now = now - q.top() + A[i].t1,q.pop(),q.push(A[i].t1);  
	}  
	printf("%d\n",res);  
	return 0;  
}  
```  
  
### P5303 [GXOI/GZOI2019]逼死强迫症  
  
一眼动态规划，然而数据范围奇大，转而想容斥……无果。  
其实是矩阵快速幂？  
考虑把动态规划过程用矩阵加速。题解大部分用到了斐波那契数列，这里我写了一种纯动态规划的解法。  
如何定义状态、转移？可以发现，一行只有单格砖是否出现这个状态，那么稍一思考我们认为有四种状态。  
  
$f(i,0,0)$ 代表都没出现，$f(i,0,1)$ 与 $f(i,1,0)$ 分别代表上下出现，$f(i,1,1)$ 上下都出现。  
  
考虑长度为二的，可以一根竖着，也可以两根横着，有：  
  
$$   f(i,0,0) = f(i-1,0,0) + f(i-2,0,0)  $$  
  
小方块纵向挨着的位置究竟放不放？我们人为定义，与小方块同横坐标为起点向右有一条长方块。  
  
然后，可以发现，放完一个小格子，之后只能横着放长格子了。  
  
$$   f(i,1,0) = f(i-2,1,0) + f(i-1,0,0)  $$  
  
$$  f(i,0,1) = f(i-2,0,1) + f(i-1,0,0)  $$  
  
最后就是合并的情况，可以发现，两个小格子都放完之后，与没放过小格子是一样的。  
两个小格子可能在同一行，可能在不同行。因此这部分转移会比较恶心。  
首先是自由放置长格子的方案：  
  
$$  f(i,1,1) += f(i-1,1,1) + f(i-2,1,1)  $$  
  
然后是同行转移的方案：  
  
$$  f(i,1,1) += f(i-3,1,0) + f(i-3,0,1)  $$  
  
然后是跨行转移的方案：  
  
$$  f(i,1,1) += f(i-2,1,0) + f(i-2,0,1)  $$  
  
真不错！每个状态都只跟前三项有关！接下来我们来汇总一下转移方程。  
  
观察一下，$f(i,1,0) = f(i,0,1)$ 显然是恒成立的，所以可以进一步化简。重新定义 $f(i,0)$ 为无小格子，$f(i,1)$ 为一个小格子，$f(i,2)$ 为两个小格子。  
  
$$  f(i,0) = f(i-1,0) + f(i-2,0)  $$  
  
$$  f(i,1) = f(i-2,1) + f(i-1,0)  $$  
  
$$  f(i,2) = f(i-1,2) + f(i-2,2) + 2 \times f(i-3,1) + 2 \times f(i-2,1)  $$  
  
是不是混乱不堪？其实需要维护的，只有 $i-1$ 与 $i-2$ 与 $i-3$ 的位置的状态就够了！  
一共 $9$ 个状态，再把转移矩阵写出即可。  
  
$$\begin{vmatrix}f(i-3,0) & f(i-3,1) & f(i-3,2) & f(i-2,0) & f(i-2,1) & f(i-2,2) & f(i-1,0) & f(i-1,1) & f(i-1,2)\end{vmatrix} \times   \begin{vmatrix}   0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\\\  0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 2 \\\\  0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \\\\  1 & 0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 \\\\  0 & 1 & 0 & 0 & 0 & 0 & 0 & 1 & 2 \\\\  0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 & 1 \\\\  0 & 0 & 0 & 1 & 0 & 0 & 1 & 1 & 0 \\\\  0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 \\\\  0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 & 1 \\\\  \end{vmatrix} =   \begin{vmatrix}f(i-2,0) & f(i-2,1) & f(i-2,2) & f(i-1,0) & f(i-1,1) & f(i-1,2) & f(i,0)& f(i,1) & f(i,2)\end{vmatrix}  $$  
  
虽然矩阵比较大，但不算太难写。  
时间复杂度 $O(729 \times T \times \log n)$.  
  
```cpp  
#include <cstdio>  
#include <cstring>  
using namespace std;  
const int mod = 1000000007;  
int T,n;  
inline int add(int x,int y){int res = x + y;return res >= mod ? res - mod : res;}  
inline int mul(int x,int y){return 1ll * x * y % mod;}  
const int maxn = 10;  
struct matrix  
{  
	int n,m,a[maxn][maxn];  
	matrix(){memset(a,0,sizeof(a));}  
	matrix operator*(const matrix &b)  
	{  
		matrix c;  
		c.n = n,c.m = b.m;  
		for(int i = 1;i<=n;++i) for(int j = 1;j<=m;++j) for(int k = 1;k<=b.m;++k) c.a[i][k] = add(c.a[i][k],mul(a[i][j],b.a[j][k]));  
		return c;  
	}  
};  
int f[4][3];  
int main()  
{  
	scanf("%d",&T);  
	f[1][0] = 1,f[1][1] = 1,f[2][0] = 2,f[2][1] = 1,f[3][0] = 3,f[3][1] = 3,f[3][2] = 2;  
	while(T--)  
	{  
		scanf("%d",&n);  
		if(n <= 3) {printf("%d\n",f[n][2]);continue;}  
		matrix O;  
		O.n = 1,O.m = 9;  
		O.a[1][1] = f[1][0],O.a[1][2] = f[1][1],O.a[1][3] = f[1][2];  
		O.a[1][4] = f[2][0],O.a[1][5] = f[2][1],O.a[1][6] = f[2][2];  
		O.a[1][7] = f[3][0],O.a[1][8] = f[3][1],O.a[1][9] = f[3][2];  
		matrix P;  
		P.n = 9,P.m = 9;  
		P.a[2][9] = 2,P.a[4][1] = 1,P.a[4][7] = 1,P.a[5][2] = 1,P.a[5][8] = 1,P.a[5][9] = 2,P.a[6][3] = 1,P.a[6][9] = 1;  
		P.a[7][4] = P.a[7][7] = P.a[7][8] = 1,P.a[8][5] = 1,P.a[9][6] = P.a[9][9] = 1;  
		n -= 3;  
		while(n)  
		{  
			if(n & 1) O = O * P;  
			P = P * P;  
			n >>= 1;  
		}  
		printf("%d\n",O.a[1][9]);  
	}  
	return 0;  
}  
```  
  
### P5460 [BJOI2016]IP 地址  
  
首先有一个差分：$[l,r]$ 范围内的变化次数，可以转化为 $[1,r] - [1,l-1]$ 的次数。  
  
转化为前缀和后，考虑按操作离线，此时只要求出 $[1,x]$ 中，在线询问的 ip 地址的变化次数即可。  
对规则建字典树，维护 $cnt(x)$ 代表以 $x$ 结尾的串的变化次数。分别考虑增加、删除对  $cnt$ 的影响。  
增加一条规则，会让它的子树内除了原本有匹配规则的节点外所有节点的 $cnt(x)$ 全部 $+1$，因为优先匹配更长的前缀。  
删除一条规则，思考一下，也是一样的。  
可以用一个懒标记维护一下。  
还有题目的扯淡描述，询问的是 $[a + 1,b]$ 区间。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <vector>  
using namespace std;  
const int maxn = 1e5 + 100;  
int n,m;  
int opt[maxn],ANS[maxn];  
char opts[maxn][34], query[maxn][34];  
vector<int> add[maxn],del[maxn];  
int son[maxn * 33][2],cnt[maxn * 33],ed[maxn * 33],tag[maxn * 33],ind;  
inline void pushdown(int x)  
{  
	if(!son[x][0]) son[x][0] = ++ind;  
	if(!son[x][1]) son[x][1] = ++ind;  
	if(!tag[x]) return;  
	if(!ed[son[x][0]]) cnt[son[x][0]] += tag[x],tag[son[x][0]] += tag[x];  
	if(!ed[son[x][1]]) cnt[son[x][1]] += tag[x],tag[son[x][1]] += tag[x];  
	tag[x] = 0;  
}  
void modify(char *s,int val)  
{  
	int u = 0;  
	for(;*s != '\0';++s) pushdown(u), u = son[u][*s - '0'];  
	ed[u] += val,tag[u]++,cnt[u]++;  
}  
int ask(char *s)  
{  
	int u = 0;  
	for(;*s != '\0';++s) pushdown(u),u = son[u][*s - '0'];  
	return cnt[u];  
}  
int main()  
{  
	scanf("%d %d",&n,&m);  
	for(int i = 1;i<=n;++i)  
	{  
		static char t[10];  
		scanf("%s %s",t,opts[i] + 1);  
		if(t[0] == 'A') opt[i] = 1; else opt[i] = -1;  
	}  
	for(int i = 1,a,b;i<=m;++i)  
	{  
		scanf("%s %d %d",query[i] + 1,&a,&b);  
		del[a].push_back(i),add[b].push_back(i);  
	}  
	for(int i = 1;i<=n;++i)  
	{  
		modify(opts[i] + 1,opt[i]);  
		for(vector<int>::iterator it = add[i].begin();it!=add[i].end();++it) ANS[*it] += ask(query[*it] + 1);  
		for(vector<int>::iterator it = del[i].begin();it!=del[i].end();++it) ANS[*it] -= ask(query[*it] + 1);  
	}  
	for(int i = 1;i<=m;++i) printf("%d\n",ANS[i]);  
	return 0;  
}  
```  
  
### P6623 [省选联考 2020 A 卷] 树  
  
维护每个子树内，点的权值加点的深度的异或和？  
  
可以发现，每次 $d(c_i,x)$ 都只会加一。  
维护关于子树的信息，想到线段树合并。由于信息与位有关，考虑 01trie 这种类似线段树的东西，那么就可以考虑 01trie 合并。  
考虑全局加一怎么处理，其实每次加一，定义根节点为二进制末尾，从根节点开始一路交换左右儿子，然后递归进入新的 $0$ 中，相当于加一之后一路进位。  
  
当然，进位后还要继承一下原先的节点值。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
const int maxn = 6e5 + 100;  
int n,w[maxn];  
int fa[maxn];  
struct node  
{  
	int to,next;  
}E[maxn];  
int head[maxn],tot;  
inline void add(int x,int y){E[++tot].to = y,E[tot].next = head[x],head[x] = tot;}  
int sz[maxn],hson[maxn];  
int root[maxn],son[maxn*4][2],cnt[maxn*4],cntsum[maxn*4],val[maxn*4],ind;  
int s[maxn * 4],top;  
inline int newnode(){return top > 0 ? s[top--] : ++ind;}  
inline void del(int x){ if(x) son[x][0] = son[x][1] = cnt[x] = cntsum[x] = val[x] = 0,s[++top] = x; }  
inline void pushup(int p)  
{  
	if(son[p][0] && son[p][1])   
	{  
		val[p] = ((val[son[p][0]] ^ val[son[p][1]]) << 1) ^ (cntsum[son[p][1]] & 1);  
		cntsum[p] = cntsum[son[p][0]] + cntsum[son[p][1]] + cnt[p];  
	}  
	else if(son[p][0]) val[p] = val[son[p][0]] << 1,cntsum[p] = cnt[p] + cntsum[son[p][0]];  
	else val[p] = (val[son[p][1]] << 1) ^ (cntsum[son[p][1]] & 1),cntsum[p] = cntsum[son[p][1]] + cnt[p];  
}  
void ins(int u,int t)  
{  
	if(!t) return (void)(cnt[u]++, cntsum[u]++);  
	if(!son[u][t&1]) son[u][t&1] = newnode();  
	ins(son[u][t&1],t>>1);  
	pushup(u);  
}  
void addall(int u)  
{  
	std::swap(son[u][0],son[u][1]);  
	if(son[u][0]) addall(son[u][0]);  
	if(!son[u][1]) son[u][1] = newnode();   
	cnt[son[u][1]] += cnt[u],cntsum[son[u][1]] += cnt[u],cnt[u] = 0;  
	pushup(u);  
}  
void merge(int &x,int y)  
{  
	if(!x || !y) return (void)(x += y);  
	merge(son[x][0],son[y][0]),merge(son[x][1],son[y][1]),cnt[x] += cnt[y],pushup(x),del(y);  
}  
int ANS[maxn];  
void pre(int u)  
{  
	sz[u] = 1;  
	for(int p = head[u];p;p=E[p].next)  
	{  
		int v = E[p].to;  
		pre(v),sz[u] += sz[v],hson[u] = sz[v] > sz[hson[u]] ? v : hson[u];  
	}  
}  
void dfs(int u)  
{  
	if(hson[u]) dfs(hson[u]),merge(root[u],root[hson[u]]);  
	for(int p = head[u];p;p=E[p].next)  
	{  
		int v = E[p].to;  
		if(v == hson[u]) continue;  
		dfs(v);  
		merge(root[u],root[v]);  
	}  
	addall(root[u]);  
	if(w[u]) ins(root[u],w[u]); else   
	{  
		if(!son[root[u]][0]) son[root[u]][0] = newnode();  
		cnt[son[root[u]][0]]++,cntsum[son[root[u]][0]]++;  
		pushup(root[u]);  
	}  
  
	ANS[u] = val[root[u]];  
}  
int main()  
{  
	scanf("%d",&n);  
	for(int i = 1;i<=n;++i) scanf("%d",w + i);  
	for(int i = 2;i<=n;++i) scanf("%d",fa + i),add(fa[i],i);  
	for(int i = 1;i<=n;++i) root[i] = newnode();  
	ind = n;  
	pre(1),dfs(1);  
	//for(int i = 1;i<=n;++i) printf("%d\n",ANS[i]);  
	long long res = 0;  
	for(int i = 1;i<=n;++i) res += ANS[i];  
	printf("%lld\n",res);  
	return 0;  
}  
```  
  
### P3201 [HNOI2009] 梦幻布丁  
  
考虑对每一个颜色建立一棵动态开点线段树，上线段树合并即可。  
如何处理当前一共有多少段颜色呢？考虑一次更改操作对答案的贡献。  
可以用线段树维护分隔连续段的段数，每次合并前后更新即可。  
  
维护左右端点是否有当前颜色。  
似乎还可以用链表启发式合并来做，时间复杂度同样是 $O(n \log n)$，  
  
```cpp  
#include <cstdio>  
const int maxn = 1e5 + 100;  
const int maxm = 1e6 + 100;  
int n,m,a[maxn],vis[maxm],res;  
int root[maxm],L[maxn * 20],R[maxn * 20],sum[maxn * 20],lval[maxn * 20],rval[maxn * 20],ind;  
int s[maxn * 20],top;  
inline int newnode(){return top > 0 ? s[top--] : ++ind;}  
inline void del(int x){if(x) L[x] = R[x] = sum[x] = lval[x] = rval[x] = 0,s[++top] = x;}  
inline void pushup(int p)  
{  
	sum[p] = sum[L[p]] + sum[R[p]],lval[p] = lval[L[p]],rval[p] = rval[R[p]];  
	if(rval[L[p]] && lval[R[p]]) --sum[p];  
  
}  
void modify(int l,int r,int &p,int pos)  
{  
	if(!p) p = newnode();  
	if(l == r) return (void)(lval[p] = rval[p] = sum[p] = 1);  
	int mid = (l + r) >> 1;  
	if(pos <= mid) modify(l,mid,L[p],pos); else modify(mid + 1,r,R[p],pos);  
	pushup(p);  
}  
void merge(int &x,int y)  
{  
	if(!x || !y) return (void)(x += y);  
	merge(L[x],L[y]),merge(R[x],R[y]),pushup(x),del(y);  
}  
int main()  
{  
	scanf("%d %d",&n,&m);  
	for(int i = 1;i<=n;++i) scanf("%d",a + i),modify(1,n,root[a[i]],i);  
	for(int i = 1;i<=n;++i) if(!vis[a[i]]) res += sum[root[a[i]]],vis[a[i]] = 1;  
	for(int i = 1,opt,x,y;i<=m;++i)  
	{  
		scanf("%d",&opt);  
		if(opt == 1)  
		{  
			scanf("%d %d",&x,&y);  
			if(x == y) continue;  
			res -= sum[root[x]] + sum[root[y]];  
			merge(root[y],root[x]),root[x] = newnode();  
			res += sum[root[y]];  
		}  
		else printf("%d\n",res);  
	}  
	return 0;  
}  
```  
  
### P5415 [YNOI2019]游戏  
  
~~YnOI 不考分块了~~  
  
这似乎是一道货真价实的云南 OI 题？  
  
初始时第 $k$ 个人最终获胜的概率，加上奇小的数据范围，感觉很是神奇。  
  
考虑动态规划。既然有一个所谓的等待队列，那么考虑将这个东西定义到状态里。  
  
定义 $f(i,j)$ 为初始时第 $k$ 个人在等待队列的第 $i$ 名，而当前的擂主连胜 $j$ 局的情况，此时初始第 $k$ 个人获胜的概率。  
  
当这个人不参与游戏时，每轮游戏它可以稳定前进三名。而擂主可能输，可能赢。  
  
那么有：  
  
$$  f(i,j) = \dfrac{1}{4} \times f(i-3,j+1) + \dfrac{3}{4} \times f(i-3,1)  $$  
  
考虑下边界条件，任意一个人连胜 $m$ 局后，游戏就结束了，只要知道这个人是不是他就可以了。  
  
$$  f(1,m) = 1  $$  
  
$$  \forall  2 \le i \le n,f(i,m) = 0  $$  
  
接下来还有他参与游戏的情况。讨论输赢之后，还要讨论获胜的人在他前方还是后方。  
  
如果他获胜，那么他成为擂主。如果原先的擂主获胜，按照编号插入尾部。如果非原先擂主获胜，那么原先擂主在尾部也要排在他的前面。  
  
$$  f(2,j) = \dfrac{1}{4} \times f(1,1) + \dfrac{1}{4} \times f(n-2,j + 1) + \dfrac{1}{2} \times f(n-1,1)  $$  
  
$$  f(3,j) = \dfrac{1}{4} \times f(1,1) + \dfrac{1}{4} \times f(n-1,j + 1) + \dfrac{1}{4} \times f(n-1,1) + \dfrac{1}{4} \times f(n,1)  $$  
  
$$  f(4,j) = \dfrac{1}{4} \times f(1,1) + \dfrac{1}{4} \times f(n,j + 1) + \dfrac{1}{2} \times f(n,1)  $$  
  
再讨论他是擂主的情况：  
  
$$  f(1,j) = \dfrac{1}{4} \times f(1,j + 1) + \dfrac{3}{4} \times f(n-2,1)  $$  
  
这下转移式就列完了，最后答案就是 $f(k,0)$，此时考虑怎么转移。  
  
这一大团东西，看上去就成环了，考虑高斯消元。每个状态都有一个转移方程，那么解这个线性多元方程组即可。  
  
一共 $n \times (m + 1)$ 个状态，时间复杂度 $O(n^3m^3)$.  
  
还有一个鬼畜的小细节：需要使用 $+=$ 而非 $=$，因为可能有从相同状态转移。例如 $j=0$ 时。  
  
```cpp  
#include <cstdio>  
#include <cstring>  
#include <cmath>  
#include <algorithm>  
using namespace std;  
const int maxn = 25;  
int T,n,m,k;  
int f[maxn][maxn],row,col;  
long double a[maxn*maxn][maxn*maxn];  
int main()  
{  
	scanf("%d",&T);  
	while(T--)  
	{  
		col = row = 0;  
		scanf("%d %d %d",&n,&m,&k);  
		memset(a,0,sizeof(a));  
		for(int i = 1;i<=n;++i) for(int j = 0;j<=m;++j) f[i][j] = ++col;  
		++col;//constant column.  
		a[++row][f[1][m]] = 1.0,a[row][col] = 1.0;  
		for(int i = 2;i<=n;++i) a[++row][f[i][m]] = 1.0,a[row][col] = 0.0;  
		for(int i = 5;i<=n;++i)  
			for(int j = 0;j<m;++j)  
				a[++row][f[i][j]] = 1.0,a[row][f[i-3][j+1]] += -0.25,a[row][f[i-3][1]] += -0.75;//!!!  
		for(int j = 0;j < m; ++j)   
		{  
			a[++row][f[1][j]] = 1.0,a[row][f[1][j+1]] += -0.25,a[row][f[n-2][1]] += -0.75;  
			a[++row][f[2][j]] = 1.0,a[row][f[1][1]] += -0.25,a[row][f[n-2][j+1]] += -0.25,a[row][f[n-1][1]] += -0.5;  
			a[++row][f[3][j]] = 1.0,a[row][f[1][1]] += -0.25,a[row][f[n-1][j+1]] += -0.25,a[row][f[n-1][1]] += -0.25,a[row][f[n][1]] += -0.25;  
			a[++row][f[4][j]] = 1.0,a[row][f[1][1]] += -0.25,a[row][f[n][j+1]] += -0.25,a[row][f[n][1]] += -0.5;  
		}  
		//for(int i = 1;i<=row;puts(""),++i) for(int j = 1;j<=col;++j) printf("%.2f ",a[i][j]);  
		for(int i = 1;i<=row;++i)  
		{  
			int maxx = i;  
			for(int j = i + 1;j<=row;++j) if(fabs(a[j][i]) > fabs(a[maxx][i])) maxx = j;  
			if(maxx != i) for(int j = 1;j<=col;++j) std::swap(a[i][j],a[maxx][j]);  
			for(int j = 1;j<=row;++j)  
			{  
				if(i == j) continue;  
				double t = a[j][i] / a[i][i];  
				for(int k = 1;k<=col;++k) a[j][k] -= t * a[i][k];  
			}  
			/*  
			printf("gauss:%d\n",i);  
			for(int i = 1;i<=row;puts(""),++i) for(int j = 1;j<=col;++j) printf("%.2f ",a[i][j]);  
			puts("");  
			*/  
		}  
		printf("%.6Lf\n",a[f[k][0]][col] / a[f[k][0]][f[k][0]]);  
	}  
	return 0;  
}  
```  
  
### P6773 [NOI2020]命运  
  
我会大暴力！对每条边枚举对应状态，预期得分 16pts.  
  
可以发现潜在的人生经历总是一条祖先到儿子的链，那么当若干个限制条件被施加，最严格的限制是上端点最深的那个限制。  
  
可以定义状态：$f(u,i)$ 代表以 $u$ 为根的子树中，未被满足的最深的询问上端点的深度为 $i$ 时的方案数。  
  
对于转移，其实就是要枚举对应边的 $0/1$ 状态。  
  
首先有边界状态：$\forall i \ge dep(u),f(u,i) = 0$，因为再往上不可能满足这些询问的限制了。  
  
再人为定义 $f(u,0)$ 为子树内限制完全在子树内满足，没有未满足的限制时的方案数。  
  
$$  f(u,i)' = \sum \limits _{j=0}^{i} f(v,j) \times f(u,i) + \sum \limits _{j=0}^{i-1} f(u,j) \times f(v,i) + \sum \limits _{j=0}^{dep(u)} f(v,j) \times f(u,i)  $$  
  
分别对应：选择 $0$ 边且子树不影响当前限制、选择 $0$ 边且子树影响当前限制、选择 $1$ 边的情况。  
注意第一二种情况的边界，当 $j=i$ 时只能统计一个。  
  
这个东西看起来可以前缀和优化一下。  
  
$$  f(u,i)' = sum(v,i) \times f(u,i) + sum(u,i-1) \times f(v,i) + sum(v,dep(u)) \times f(u,i)  $$  
  
合并一下：  
  
$$  f(u,i)' = f(u,i) \times (sum(v,i) + sum(v,dep(u)) + f(v,i) \times sum(u,i-1)  $$  
  
这个 $sum(u,i-1)$ 看似有点鬼畜，但其实不复杂。因为转移到 $f(u,i)'$ 时一定知道 $f(u,i)$ 的值，那么这个就不难了。  
  
这个东西怎么做？考虑维护一棵存放 $f(u,i)$ 的线段树，在树上跑线段树合并，在合并的过程中，用线段树二分的手法去维护前缀和。  
  
还有一个 $sum(v,dep(u))$ 鹤立鸡群，直接查询出来。  
  
在合并操作中，做区间加、区间乘，注意标记顺序即可。  
  
还需要注意标记下放，可能会破坏线段树合并的复杂度，因为下放产生的新节点也会合并进去，随后不断产生复杂度代价把整棵树跑满。  
  
由于是乘法标记，因此空节点可以直接不下放。  
  
如果是通用标记，可以在合并过程中判断是空节点时，下放标记但不递归。  
  
```cpp  
#include <algorithm>  
#include <cctype>  
#include <cstdio>  
const int maxn = 1e6 + 100;  
const int mod = 998244353;  
inline int add(int x, int y) { int res = x + y; return res >= mod ? res - mod : res; }  
inline int mul(int x, int y) { return 1ll * x * y % mod; }  
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
struct node  
{  
    int to, next;  
} E[maxn << 2];  
int Eh[maxn], Gh[maxn], tot;  
inline void addE(int x, int y) { E[++tot].next = Eh[x], Eh[x] = tot, E[tot].to = y; }  
inline void addG(int x, int y) { E[++tot].next = Gh[x], Gh[x] = tot, E[tot].to = y; }  
int n, m, dep[maxn], up[maxn], size[maxn], son[maxn], maxdep;  
void dfs1(int u, int fa)  
{  
    dep[u] = dep[fa] + 1;  
    for (int p = Gh[u]; p; p = E[p].next) up[u] = std::max(up[u], dep[E[p].to]);  
    size[u] = 1;  
    for (int p = Eh[u]; p; p = E[p].next)  
    {  
        int v = E[p].to;  
        if (v == fa) continue;  
        dfs1(v, u), size[u] += size[v];  
        if (size[v] > size[son[u]]) son[u] = v;  
    }  
}  
namespace seg  
{  
int root[maxn], L[maxn * 10], R[maxn * 10], sum[maxn * 10], multag[maxn * 10], s[maxn * 10], top, ind;  
inline int newnode()  
{  
    int id = top > 0 ? s[top--] : ++ind;  
    L[id] = R[id] = sum[id] = 0, multag[id] = 1;  
    return id;  
}  
inline void delnode(int x) { if (x) s[++top] = x; }  
inline void pushdown(int p)  
{  
    if (multag[p] != 1)  
    {  
        if (L[p])  
        {  
            sum[L[p]] = mul(sum[L[p]], multag[p]);  
            multag[L[p]] = mul(multag[L[p]], multag[p]);  
        }  
        if (R[p])  
        {  
            sum[R[p]] = mul(sum[R[p]], multag[p]);  
            multag[R[p]] = mul(multag[R[p]], multag[p]);  
        }  
        multag[p] = 1;  
    }  
}  
inline void pushup(int p) { sum[p] = add(sum[L[p]], sum[R[p]]); }  
void modify(int l, int r, int& p, int pos, int k)  
{  
    if (!p) p = newnode();  
    if (l == r) return (void)(sum[p] += k);  
    int mid = (l + r) >> 1;  
    pushdown(p);  
    if (pos <= mid) modify(l, mid, L[p], pos, k); else modify(mid + 1, r, R[p], pos, k);  
    pushup(p);  
}  
int ask(int l, int r, int p, int ll, int rr)  
{  
    if (!p) return 0;  
    if (l >= ll && r <= rr) return sum[p];  
    int mid = (l + r) >> 1, res = 0;  
    pushdown(p);  
    if (ll <= mid) res = ask(l, mid, L[p], ll, rr);  
    if (rr > mid) res = add(res, ask(mid + 1, r, R[p], ll, rr));  
    return res;  
}  
int merge(int l, int r, int x, int y, int& xsum, int& ysum)  //ysum is originally depval  
{  
    if (!x && !y) return 0;  
    if (!x || !y)  
    {  
        if (!x)  
        {  
            ysum = add(ysum, sum[y]);  
            sum[y] = mul(sum[y], xsum), multag[y] = mul(multag[y], xsum);  
            return y;  
        }  
        xsum = add(xsum, sum[x]);  
        sum[x] = mul(sum[x], ysum), multag[x] = mul(multag[x], ysum);  
        return x;  
    }  
    if (l == r)  
    {  
        ysum = add(ysum, sum[y]);  
        int oldx = sum[x];  
        sum[x] = add(mul(sum[y], xsum), mul(sum[x], ysum));  
        xsum = add(xsum, oldx);  
        delnode(y);  
        return x;  
    }  
    int mid = (l + r) >> 1;  
    pushdown(x), pushdown(y);  
    L[x] = merge(l, mid, L[x], L[y], xsum, ysum);  
    R[x] = merge(mid + 1, r, R[x], R[y], xsum, ysum);  
    pushup(x), delnode(y);  
    return x;  
}  
}  // namespace seg  
void dfs2(int u, int fa)  
{  
    seg::modify(0, maxdep, seg::root[u], up[u], 1);  
    if (son[u])  
    {  
        dfs2(son[u], u);  
        int xsum = 0, ysum = seg::ask(0, maxdep, seg::root[son[u]], 0, dep[u]);  
        seg::root[u] = seg::merge(0, maxdep, seg::root[u], seg::root[son[u]], xsum, ysum);  
    }  
    for (int p = Eh[u]; p; p = E[p].next)  
    {  
        int v = E[p].to;  
        if (v == fa || v == son[u]) continue;  
        dfs2(v, u);  
        int xsum = 0, ysum = seg::ask(0, maxdep, seg::root[v], 0, dep[u]);  
        seg::root[u] = seg::merge(0, maxdep, seg::root[u], seg::root[v], xsum, ysum);  
    }  
}  
signed main()  
{  
    read(n);  
    for (int i = 1, a, b; i < n; ++i) read(a), read(b), addE(a, b), addE(b, a);  
    read(m);  
    for (int i = 1, a, b; i <= m; ++i) read(a), read(b), addG(a, b), addG(b, a);  
    dfs1(1, 0);  
    for (int i = 1; i <= n; ++i) maxdep = std::max(maxdep, dep[i]);  
    dfs2(1, 0);  
    printf("%d\n", seg::ask(0, maxdep, seg::root[1], 0, 0));  
    return 0;  
}  
```  
  
### P3750 [六省联考 2017]分手是祝愿  
  
神仙题。完全没有思路。抄题解。  
  
一个数的约数是 $\log n$ 级别的，那么对一个数的操作可以直接上暴力模拟。  
  
直接埃氏筛预处理约数，调和级数复杂度。  
  
然后引出神秘结论：  
  
每个开关的效果都是独一无二的。不存在开关组合，使得该组合执行后能得到某开关的效果。  
  
从大到小，遇到亮的就消除，这个贪心就能获取最少步数。因为：如果它不是此时消除，就只能用更大的数消除，而原本是亮的更大的数已经消去，将灭的更大的数点亮后，大数又无法消除。而每个开关，最多操作一次，因为两次相当于没操作。  
  
那么模拟一遍，就知道最少按几个键了。  
  
知道最少按几个键之后，定义 $f(i)$ 为还需要按 $i$ 个键的期望步数。初始，有 $f(num) = 0$.  
  
$$  f(i) = \dfrac{i}{n} \times (f(i-1) + 1) + \dfrac{n-i}{n} \times (f(i+1) + 1)  $$  
  
~~我会高斯消元！~~显然高斯消元是无法通过这么大的范围的。  
  
那么按照套路，换一种方法，定义 $f(i)$ 为按掉第 $i$ 个需要的键的期望步数，即从剩余 $i$ 个键状态转移到剩余 $i-1$ 个键状态的期望步数。初始时有 $f(n) = 1$.  
  
$$  f(i) = \dfrac{i}{n} + \dfrac{n-i}{n} \times (f(i) + f(i+1) + 1)  $$  
  
这个东西就是可做的了，化简一下有：  
  
$$  f(i) = \dfrac{n-i}{i} \times f(i+1) + \dfrac{n}{i}  $$  
  
最后统计答案，判断一下 $num$ 与  $k$ 的关系，然后累加即可。最后再莫名其妙地乘一个阶乘。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <vector>  
using namespace std;  
const int maxn = 1e5 + 100;  
const int mod = 100003;  
inline int add(int x,int y){int res = x + y;return res >= mod ? res - mod : res;}  
inline int mul(int x,int y){return 1ll * x * y % mod;}  
vector<int> frac[maxn];  
int n,k,a[maxn],f[maxn],inv[maxn];  
int main()  
{  
	scanf("%d %d",&n,&k);  
	for(int i = 1;i<=n;++i) scanf("%d",a + i);  
	for(int j = 1;j<=n;++j) for(int i = 1;i<=n;i+=j) frac[i].push_back(j);  
	int num = 0;  
	for(int i = n;i;--i) if(a[i])   
	{  
		for(vector<int>::iterator it = frac[i].begin();it!=frac[i].end();++it) a[*it] ^= 1;  
		++num;  
	}  
	if(k >= num)   
	{  
		for(int i = 1;i<=n;++i) num = mul(num,i);  
		printf("%d\n",num);  
	}  
	else  
	{  
		inv[1] = 1;  
		for(int i = 2;i<=n;++i) inv[i] = mul(mod - mod / i,inv[mod % i]);  
		f[n] = 1;  
		for(int i = n - 1;i;--i) f[i] = add(mul(mul(n-i,inv[i]),f[i+1]),mul(n,inv[i]));  
		for(int i = 1;i<=n;++i) printf("%d %d\n",i,f[i]);  
		int res = k;  
		for(int i = num;i > k;--i) res = add(res,f[i]);  
		for(int i = 1;i<=n;++i) res = mul(res,i);  
		printf("%d\n",res);  
	}  
	return 0;  
}  
```  
  
### P3749 [六省联考 2017]寿司餐厅  
  
最大权闭合子图。  
  
考虑将 $d(i,j)$ 的收益抽象成带权点，可以发现，选择 $d(i,j)$ 则一定选择 $d(i+1,j)$ 与 $d(i,j-1)$，可以依据此连边。  
  
此外，代价的贡献可以拆成选择某种类型的物品一次性支付一定代价，随后每选择一个该种物品付出一定代价。  
将物品种类抽象成带权点，选择 $d(i,i)$ 则一定选择对应的物品种类。至于单个物品代价，可以直接对 $d(i,i)$ 做减法。  
  
使用最小割求解。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
const int maxn = 110;  
const int maxm = 1e6;  
struct node  
{  
	int to,next,cap;  
}E[maxm];  
int head[maxm],tot = 1;  
inline void add(int x,int y,int cap)  
{  
	E[++tot].next = head[x],head[x] = tot,E[tot].to = y,E[tot].cap = cap;  
	E[++tot].next = head[y],head[y] = tot,E[tot].to = x,E[tot].cap = 0;  
}  
int n,m,a[maxn],d[maxn][maxn],id[maxn][maxn],type[maxn*10],S,T,cnt,maxtype;  
const int inf = 1 << 28;  
int dep[maxm],cur[maxm];  
bool bfs()  
{  
	static int q[maxm],qh,qt;  
	for(int i = 1;i<=cnt;++i) dep[i] = 0;  
	qh = 1,q[qt = 1] = S,dep[S] = 1,cur[S] = head[S];  
	while(qt >= qh)  
	{  
		int u = q[qh++];  
		if(u == T) return 1;  
		for(int p = head[u],v;p;p=E[p].next)   
			if(E[p].cap && !dep[v = E[p].to]) dep[v] = dep[u] + 1,q[++qt] = v,cur[v] = head[v];  
	}  
	return 0;  
}  
long long dfs(int u,long long flow)  
{  
	if(!flow || u == T) return flow;  
	long long vflow = 0,sumflow = 0;  
	for(int &p = cur[u];p;p=E[p].next)  
	{  
		int v = E[p].to;  
		if(E[p].cap && dep[v] == dep[u] + 1 && (vflow = dfs(v,std::min(flow,1ll * E[p].cap))))  
		{  
			flow -= vflow,sumflow += vflow,E[p].cap -= vflow,E[p^1].cap += vflow;  
			if(flow == 0) break;  
		}  
	}  
	return sumflow;  
}  
int main()  
{  
	scanf("%d %d",&n,&m);  
	S = ++cnt,T = ++cnt;  
	for(int i = 1;i<=n;++i) scanf("%d",a + i),maxtype = std::max(maxtype,a[i]);  
	for(int i = 1;i<=n;++i) for(int j = i;j<=n;++j) scanf("%d",d[i] + j),id[i][j] = ++cnt;  
	for(int i = 1;i<=maxtype;++i) type[i] = ++cnt;  
	long long sum = 0;  
	for(int i = 1;i<=maxtype;++i) add(type[i],T,m * i * i);  
	for(int i = 1;i<=n;++i) add(id[i][i],type[a[i]],inf);  
	for(int i = 1;i<=n;++i) d[i][i] -= a[i];  
	for(int i = 1;i<=n;++i)   
	{  
		for(int j = i;j<=n;++j)   
		{  
			if(d[i][j] > 0)  add(S,id[i][j],d[i][j]),sum += d[i][j];  
			else add(id[i][j],T,-d[i][j]);  
			if(id[i][j-1] > 0) add(id[i][j],id[i][j-1],inf);  
			if(id[i+1][j] > 0) add(id[i][j],id[i+1][j],inf);  
		}  
	}  
	long long sumflow = 0;  
	while(bfs()) sumflow += dfs(S,1ll << 60);  
	printf("%lld\n",sum - sumflow);  
	return 0;  
}  
```  
  
### P4159 [SCOI2009] 迷路  
  
定义 $f(u,i)$ 为在时刻 $i$ 位于位置 $t$ 的方案数。  
  
$$  
f(v,i+1) = \prod f(u,i)  
$$  
  
然后使用美食家的套路，拆点来做边权 $w > 1$ 的情况。  
  
```cpp  
#include <cstdio>  
#include <cstring>  
#include <algorithm>  
using namespace std;  
const int maxn = 110;  
const int mod = 2009;  
inline int add(int x,int y){int res = x + y;return res >= mod ? res - mod : res;}  
inline int mul(int x,int y){return 1ll * x * y % mod;}  
struct matrix  
{  
	int n,m;  
	int a[maxn][maxn];  
	matrix() {memset(a,0,sizeof(a));}  
	matrix operator*(const matrix b)  
	{  
		matrix c;  
		c.n = n,c.m = b.m;  
		for(int i = 1;i<=n;++i) for(int j = 1;j<=m;++j) for(int k = 1;k<=b.m;++k)   
			c.a[i][k] = add(c.a[i][k],mul(a[i][j],b.a[j][k]));  
		return c;  
	}  
};  
int n,t,cnt,id[maxn][maxn];  
matrix A,B;  
char s[maxn];  
int main()  
{  
	scanf("%d %d",&n,&t);  
	for(int i = 1;i<=n;++i) for(int j = 1;j<=9;++j) id[i][j] = ++cnt;  
	for(int i = 1;i<=n;++i) for(int j = 2;j<=9;++j) B.a[id[i][j]][id[i][j-1]] = 1;  
	for(int i = 1;i<=n;++i)  
	{  
		scanf("%s",s + 1);  
		for(int j = 1;j<=n;++j)   
			if(s[j] != '0')  
			{  
				int d = s[j] - '0';  
				B.a[id[i][1]][id[j][d]] = 1;  
			}  
	}  
  
	A.n = 1,A.m = cnt,B.n = B.m = cnt;  
	A.a[1][id[1][1]] = 1;  
	for(;t;t >>= 1)  
	{  
		if(t & 1) A = A * B;  
		B = B * B;  
	}  
	printf("%d\n",A.a[1][id[n][1]]);  
	return 0;  
}  
```  
  
### P3597 [POI2015]WYC  
  
走 $k$ 步的总路径数是可以用矩阵计算出来的，那么有一个想法：倍增处理出 $2^k$ 步的转移矩阵，然后用状态矩阵去类似倍增那样找出步数。  
  
然后同样拆点处理边权即可。  
  
有一个鬼畜的坑点：连接相同两点的两条边，也看作不同，因此需要 $+1$ 而不能直接赋值。  
  
注意一下不要溢出，可以用极大值标记的方法。  
  
```cpp  
#include <cstdio>  
#include <cstring>  
#include <algorithm>  
using namespace std;  
const int maxn = 200;  
long long k;  
inline long long add(long long x,long long y)  
{  
	if(x == -1 || y == -1) return -1;  
	long long res = x + y;  
	return res > k ? -1 : res;  
}  
inline long long mul(long long x,long long y)  
{  
	if(x == 0 || y == 0) return 0;  
	if(x == -1 || y == -1) return -1;  
	long long res = x * y;  
	return res > k ? -1 : res;  
}  
struct matrix  
{  
	int n,m;  
	long long a[maxn][maxn];  
	matrix(){memset(a,0,sizeof(a));}  
	matrix operator*(const matrix &b)  
	{  
		matrix c;  
		c.n = n,c.m = b.m;  
		for(int i = 1;i<=n;++i) for(int j = 1;j<=m;++j) for(int k = 1;k<=b.m;++k)  
			c.a[i][k] = add(c.a[i][k],mul(a[i][j],b.a[j][k]));  
		return c;  
	}  
	void output()  
	{  
		puts("");  
		for(int i = 1;i<=n;puts(""),++i) for(int j = 1;j<=m;++j) printf("%lld ",a[i][j]);  
		puts("");  
	}  
}A[100];  
int n,m,id[maxn][4],ind;  
int main()  
{  
	scanf("%d %d %lld",&n,&m,&k);  
	for(int i = 1;i<=n;++i) for(int j = 1;j<=3;++j) id[i][j] = ++ind;  
	++ind;//sum point  
	A[0].n = ind,A[0].m = ind;  
	for(int i = 1;i<=n;++i) for(int j = 2;j<=3;++j) A[0].a[id[i][j]][id[i][j-1]] = 1;  
	for(int i = 1;i<=n;++i) A[0].a[id[i][1]][ind] = 1;  
	A[0].a[ind][ind] = 1;  
	for(int i = 1,a,b,c;i<=m;++i)  
	{  
		scanf("%d %d %d",&a,&b,&c);  
		A[0].a[id[a][1]][id[b][c]] += 1;  
	}  
	int maxlog = 1;  
	matrix S;  
	S.n = 1,S.m = ind;  
	for(int i = 1;i<=n;++i) S.a[1][id[i][1]] = 1;  
	S.a[1][ind] = -n;  
	for(;;++maxlog)  
	{  
		A[maxlog] = A[maxlog-1] * A[maxlog-1];  
		matrix T = S * A[maxlog];  
		long long sum = T.a[1][ind];  
		if(sum == -1) break;  
		for(int j = 1;j<=n;++j) sum = add(sum,T.a[1][id[j][1]]);  
		if(sum == -1) break;  
		if(maxlog > 70){puts("-1");return 0;}  
	}  
	//A[0].output(),A[1].output();  
	long long res = 0;  
	for(int i = maxlog;i >= 0;--i)  
	{  
		matrix T = S * A[i];  
		long long sum = T.a[1][ind];  
		if(sum == -1) continue;  
		for(int j = 1;j<=n;++j) sum = add(sum,T.a[1][id[j][1]]);  
		if(sum == -1) continue;  
		if(sum < k) res |= 1ll << i,S = T;  
		//printf("%d %lld\n",i,sum);  
		//if(sum == -1) continue;  
	  
	}  
	//S.output();  
	printf("%lld\n",res + 1);  
	return 0;  
}  
```  
  
### P3582 [POI2015]KIN  
  
题意：$m$ 种物品，每种权值为 $w_i$ ，给一个长为 $n$ 的序列，第 $i$ 个位置是第 $a_i$ 种物品。选择一个区间 $[l,r]$，随后可以获取区间内仅出现一次的物品的权值和。  
  
最大化权值和。  
  
考虑常规套路，移动一个端点，用数据结构维护选择另一个端点的答案，看起来就很线段树。  
  
考虑左端点向左单调，当加入一个点时，它到它上次出现位置的答案会增加它的权值，而它的上次出现到它的上两次出现会减少它的权值。  
  
所以就可以动态维护了，总感觉这个东西被出烂了。  
  
区间加、全局最大值即可。  
  
第一次写忘记下推标记了，我也太蠢了。  
  
```cpp  
#include <cstdio>  
#include <cctype>  
#include <algorithm>  
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
	for(c=nc();!isdigit(c);c=nc());  
	for(;isdigit(c);c=nc()) r=r*10+c-48;  
	return r;  
}  
const int maxn = 1e6 + 100;  
int n,m,a[maxn],w[maxn],last[maxn],vis[maxn];  
long long maxx[maxn << 2],tag[maxn << 2];  
#define ls p << 1  
#define rs p << 1 | 1  
inline void pushup(int p){maxx[p] = max(maxx[ls],maxx[rs]);}  
inline void pushdown(int p)  
{  
	if(!tag[p]) return;  
	maxx[ls] += tag[p],maxx[rs] += tag[p],tag[ls] += tag[p],tag[rs] += tag[p],tag[p] = 0;  
}  
void modify(int l,int r,int p,int ll,int rr,int k)  
{  
	if(l >= ll && r <= rr) return (void)(maxx[p] += k,tag[p] += k);  
	int mid = (l + r) >> 1;  
	pushdown(p);  
	if(ll <= mid) modify(l,mid,ls,ll,rr,k);   
	if(rr > mid) modify(mid + 1,r,rs,ll,rr,k);  
	pushup(p);  
}  
signed main()  
{  
	read(n),read(m);  
	for(int i = 1;i<=n;++i) read(a[i]);  
	for(int i = 1;i<=m;++i) read(w[i]);  
	for(int i = n;i;--i)   
	{  
		if(!vis[a[i]]) last[i] = n + 1;  
		else last[i] = vis[a[i]];  
		vis[a[i]] = i;  
	}  
	long long res = 0;  
	for(int l = n;l;--l)  
	{  
		modify(1,n,1,l,last[l] - 1,w[a[l]]);  
		if(last[l] <= n) modify(1,n,1,last[l],last[last[l]] - 1,-w[a[l]]);  
		res = std::max(res,maxx[1]);  
	}  
	printf("%lld\n",res);  
	return 0;  
}  
  
```  
  
### CF1327F AND Segments  
  
首先每位是独立的，拆位后，计算每位的方案数用乘法原理。  
  
对于一个限制，拆位后，对应位要么是零要么是一。如果是零，则要求 $[L,R]$ 中至少有一个零。如果是一，则要求 $[L,R]$ 中全部是一。  
  
可以考虑动态规划？定义 $f(i,j)$ 为考虑前 $i$ 位，最后一个零在第 $j$ 位的方案数。  
  
可以发现，零限制的 $L$ 越大，限制越严格，只需要用到最严的限制即可。  
  
而一限制可以转化为对单点限制，即限制 $[L,R]$ 间的点强制选择一。  
  
记最严格的零限制的左端点为 $lim(i)$，且保证 $lim(i) \le i$：  
  
$$  \forall j < lim(i), f(i,j) = 0  $$  
  
$$  \forall lim(i) \le j < i, f(i,j) = f(i-1,j)  $$  
  
如果 $i$ 点强制选一，那么 $f(i,i) = 0$，否则有：$f(i,i) = \sum \limits _{k=lim(i-1)}^{i-1} f(i-1,j)$  
  
第一反应拿数据结构硬上，然而拆位后复杂度必须是线性，否则无法通过。  
  
有  $lim(i)$ 单调递增，可以用一个指针来维护。指针移动的同时还可以维护 $f(i,i) = \sum \limits _{k=lim(i-1)}^{i-1} f(i-1,j)$，那么一切都好起来了。  
  
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
	static char buf[bufSize],*p1 = buf,*p2 = buf;  
	return p1==p2&&(p2=(p1=buf)+fread(buf,1,bufSize,stdin),p1==p2)?EOF:*p1++;  
}  
template<typename T>  
inline T read(T &r)  
{  
	static char c;  
	r = 0;  
	for(c=nc();!isdigit(c);c=nc());  
	for(;isdigit(c);c=nc()) r=r*10+c-48;  
	return r;  
}  
const int maxn = 5e5 + 100;  
const int mod = 998244353;  
inline int add(int x,int y){int res = x + y;return res >= mod ? res - mod : res;}  
inline int mul(int x,int y)  
{  
	long long res = 1ll * x * y;  
	return res >= mod ? res % mod : res;  
}  
struct node  
{  
	int l,r,x;  
}A[maxn];  
int n,m,k,lim[maxn],f[maxn],d[maxn];  
int main()  
{  
	read(n),read(k),read(m);  
	for(int i = 1;i<=m;++i) read(A[i].l),read(A[i].r),read(A[i].x);  
	int res = 1;  
	for(int bit = 0;bit < k;++bit)  
	{  
		for(int i = 1;i<=n + 1;++i) d[i] = lim[i] = f[i] = 0;  
		for(int i = 1;i<=m;++i)   
			if((A[i].x >> bit) & 1) d[A[i].l]++,d[A[i].r+1]--;  
			else lim[A[i].r] = max(lim[A[i].r],A[i].l);  
		for(int i = 2;i<=n + 1;++i) d[i] += d[i-1], lim[i] = max(lim[i],lim[i-1]);  
		f[0] = 1;  
		int p = 0,sum = f[0];  
		for(int i = 1;i<=n + 1;++i)  
		{  
			if(!d[i]) f[i] = sum,sum = add(sum,f[i]);  
			while(p < lim[i]) sum = add(sum,mod - f[p++]);  
		}  
		//printf("%d %d\n",bit,f[n+1]);  
		res = mul(res,f[n+1]);  
	}  
	printf("%d\n",res);  
	return 0;  
}  
```  
  
### CF1039D You Are Given a Tree  
  
给一棵 $n$ 个节点的树，询问对于 $k \in [1,n]$，这棵树划分为若干条不相交的长度为 $k$ 的简单路径，能得到的最大路径数。  
  
首先明确对于固定的 $k$ 如何求出答案。  
  
对于一个以 $u$ 为根的子树，要么最长链与次长链拼接为答案做出贡献，要么让最长链向上延伸。  
  
如果拼接可以 $>k$，那么直接拼接。因为向上延伸后也最多只能做出一次贡献，并且潜力更小。  
  
那么可以 $O(nk)$ 地暴力求解，没有感觉到什么优化方法……  
  
不过有一个直观感受，当 $k$ 上升时，路径数一定快速减小，并且有长段的平台。  
  
可以发现，路径数的值域是 $[0,n]$，如果对每个值，二分出一个对应的 $k$ 的取值区间，就可以解决问题。  
  
然而会发现，对于一个值，需要 $O(n \log n)$ 的代价才能求出对应的区间，这甚至大于暴力求解的 $O(n)$，如果值域这么大反而会跑得更慢。  
  
那么一个 $k$ 的值域形式是什么？$\dfrac{n}{k}$，在 $k$ 较小时，随着 $k$ 增加，值域迅速缩小，因此可以考虑根号分治。  
  
设阙值为 $B$，对于 $k \in [1,B]$ 直接暴力，否则二分，时间复杂度 $O(nB + \dfrac{n}{B} \times n \log n)$，在 $B = \sqrt{n \log n}$ 时最优。  
  
这样可以得到一个 $O(n\sqrt{n \log n})$ 的优秀做法，虽然常数巨大，但调调块长应该能过。  
  
单次二分代价大，能不能整体二分？  
  
可以发现，随着 $k$ 单调递增，路径数单调递减，可以利用这个性质划分值域。  
  
这样可能会退化到 $O(n^2)$，然而实际上复杂度是 $O(n \sqrt{n})$  的，因为：  
  
不同路径数的取值最多有 $2 \times \sqrt{n}$ 种。对于 $k \in [1,\sqrt{n}]$，每个 $k$ 可以对应一种取值，而对于 $k > \sqrt{n}$，$\dfrac{n}{k} \in [1,\sqrt{n}]$，因此也最多有 $\sqrt{n}$ 种取值。  
  
也就是说，这个整体二分的递归树，最多只有 $\sqrt{n}$ 级别的叶子数量。每个叶子深度最多是 $\log n$，那么最多存在 $\sqrt{n} \times \log n$ 次有效递归，时间复杂度上界是 $O(n\sqrt{n} \log {n})$ 的，看起来很大，但考虑到有许多公共路径不会重复计算，因此常数很小。  
  
然而这个做法应该是严格劣于根号分治的。  
  
```cpp  
#include <cstdio>  
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
	static char buf[bufSize],*p1 = buf,*p2=buf;  
	return p1==p2&&(p2=(p1=buf)+fread(buf,1,bufSize,stdin),p1==p2)?EOF:*p1++;  
}  
template<typename T>  
inline T read(T &r)  
{  
	static char c;  
	r = 0;  
	for(c=nc();!isdigit(c);c=nc());  
	for(;isdigit(c);c=nc()) r = r*10+c-48;  
	return r;  
}  
const int maxn = 1e5 + 100;  
struct node  
{  
	int to,next;  
}E[maxn << 1];  
int head[maxn],tot;  
inline void add(int x,int y){E[++tot].next = head[x],head[x] = tot,E[tot].to = y;}  
int n;  
int nowk,num,f[maxn];  
void dfs(int u,int fa)  
{  
	int maxx = 0,secx = 0;  
	for(int p = head[u];p;p=E[p].next)  
	{  
		int v = E[p].to;  
		if(v == fa) continue;  
		dfs(v,u);  
		if(f[v] > maxx) secx = maxx,maxx = f[v];  
		else if(f[v] > secx) secx = f[v];  
	}  
	if(maxx + secx + 1 >= nowk) ++num,f[u] = 0;  
	else f[u] = maxx + 1;  
}  
inline int get(int x)  
{  
	num = 0,nowk = x,dfs(1,0);  
	return num;  
}  
int ANS[maxn];  
void solve(int l,int r,int L,int R)  
{  
	if(l > r || L > R) return;  
	if(l == r)  
	{  
		for(int i = L;i<=R;++i) ANS[i] = l;  
		return;  
	}  
	if(L == R) return (void)(ANS[L] = get(L));  
	/*  
	int mid = (L + R) >> 1,lres = get(L),rres = get(R);  
	ANS[L] = lres,ANS[R] = rres;  
	if(lres == rres)  
	{  
		for(int i = L + 1;i<R;++i) ANS[i] = lres;  
		return;  
	}  
	*/  
	int mid = (L + R) >> 1,res = get(mid);  
	ANS[mid] = res;  
	solve(res,r,L,mid - 1),solve(l,res,mid + 1,R);  
	//solve(res,lres,L + 1,mid - 1),solve(rres,res,mid + 1,R - 1);  
}  
int main()  
{  
	read(n);  
	for(int i = 1,a,b;i<n;++i) read(a),read(b),add(a,b),add(b,a);  
	solve(0,n,1,n);  
	for(int i = 1;i<=n;++i) printf("%d\n",ANS[i]);  
	return 0;  
}  
```  
  
### P3616 富金森林公园  
  
维护 $a_i \ge k$ 的连续段数，是可以用线段树维护的。  
  
然后乱搞一个树套树，但是做不了。因为无法合并 $\log$ 棵树的答案。  
  
维护 $f(i)$ 代表海拔为 $i$ 时的答案。  
  
联通块个数等于点数减去边数。  
  
考虑一个点，在 $h \le a_i$ 时都会存在，考虑一条边，在 $h \le \min(a_i,a_{i+1})$ 时都会存在。  
  
这个动态维护即可。  
  
```cpp  
#include <cstdio>  
#include <vector>  
#include <algorithm>  
using namespace std;  
const int maxn = 4e5 + 100;  
struct node  
{  
	int opt,pos,val;  
}A[maxn];  
int n,m,a[maxn],H[maxn],cnt;  
int sum[maxn];  
inline void add(int x,int k) { for(++x;x <= cnt + 10;x+=x&-x) sum[x] += k;}  
inline void addrange(int l,int r,int k){add(l,k),add(r+1,-k);}  
inline int ask(int x){int res = 0;for(++x;x>0;x-=x&-x) res += sum[x];return res;}  
inline void update(int x,int k)  
{  
	int minn = min(a[x],a[x+1]);  
	addrange(1,minn,-k);  
}  
int main()  
{  
	scanf("%d %d",&n,&m);  
	H[++cnt] = 1 << 30;  
	for(int i = 1;i<=n;++i) scanf("%d",a + i),H[++cnt] = a[i];  
	for(int i = 1;i<=m;++i)  
	{  
		scanf("%d",&A[i].opt);  
		if(A[i].opt == 1) scanf("%d",&A[i].val),H[++cnt] = A[i].val;  
		else scanf("%d %d",&A[i].pos,&A[i].val),H[++cnt] = A[i].val;  
	}  
	std::sort(H + 1,H + 1 + cnt),cnt = std::unique(H + 1,H + 1 + cnt) - H - 1;  
	for(int i = 1;i<=n;++i) a[i] = lower_bound(H + 1,H + 1 + cnt,a[i]) - H;  
	for(int i = 1;i<=m;++i) A[i].val = lower_bound(H + 1,H + 1 + cnt,A[i].val) - H;  
	for(int i = 1;i<=n;++i) addrange(1,a[i],1);  
	for(int i = 1;i<n;++i) update(i,1);  
	for(int i = 1;i<=m;++i)  
	{  
		if(A[i].opt == 1) printf("%d\n",ask(A[i].val));  
		else  
		{  
			int p = A[i].pos,v = A[i].val;  
			add(a[p] + 1,1),add(v + 1,-1);  
			update(p - 1,-1),update(p,-1);  
			a[p] = v;  
			update(p-1,1),update(p,1);  
		}  
	}  
	return 0;  
}  
```  
  
### P4643 [国家集训队]阿狸和桃子的游戏  
  
可以考虑模拟选点过程？然而因为有边权的存在，选点具有后效性。  
查看题解。  
  
把边权均摊到两个点上，即每个点都加上 $\dfrac{c}{2}$，然后贪心选择最大即可。  
  
为什么是正确的？当同一个人选择两个点时，这条边权被正确累计。当两个点分别属于两个人时，要求最大化的答案 $X + \dfrac{c}{2} - (Y + \dfrac{c}{2})$，也正确累计。  
  
这说明选择的点集的转化权值和的差等价于原来的权值和的差。  
  
然后贪心一下就好了。感觉被恶评了？原题似乎是动态 DP，改题号的时候没清空难度……  
  
不过这个 trick 还是很有趣的。如果要构造出真实的得分，可以记录选择方案，最后计算一遍。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
int n,m,w[11000],res;  
int main()  
{  
	scanf("%d %d",&n,&m);  
	for(int i = 1;i<=n;++i) scanf("%d",w + i),w[i] <<= 1;  
	for(int i = 1,a,b,c;i<=m;++i) scanf("%d %d %d",&a,&b,&c),w[a] += c,w[b] += c;  
	std::sort(w + 1,w + 1 + n);  
	for(int i = n;i;--i) res += ((i & 1) ? -1 : 1) * w[i];  
	return printf("%d\n",res >> 1),0;  
}  
```  
  
### P4317 花神的数论题  
  
首先发现 $sum(i)$ 的值域范围很小，可以考虑统计每个 $sum(i)$ 的值出现了多少次。  
  
如何统计？定义 $f(i,j)$ 为剩余 $i$ 位待定，当前 $sum = j$ 的次数，然后数位 DP 的套路即可。  
  
其实没必要钦定 $sum = j$，可以直接返回乘积。这样可以做到 $O(\log^2 n)$ 的复杂度。  
  
```cpp  
#include <cstdio>  
#include <cstring>  
#include <algorithm>  
using namespace std;  
const int mod = 10000007;  
long long f[70][70][2];  
long long n;  
long long dfs(int pos,int limit,int sum)  
{  
	if(pos == -1) return max(1,sum);  
	if(f[pos][sum][limit] != -1) return f[pos][sum][limit];  
	int up = limit ? (n >> pos) & 1 : 1;  
	long long now = 1;  
	for(int i = 0;i<=up;++i) now = now * dfs(pos - 1,limit && i == up,sum + i) % mod;  
	return f[pos][sum][limit] = now;  
}  
int main()  
{  
  
	scanf("%lld",&n);  
	int up = 0;  
	while((1ll << up) < n) ++up;  
	memset(f,-1,sizeof(f));  
	printf("%lld\n",dfs(up,1,0));  
	return 0;  
}  
```  
  
### P6198 [EER1]单调栈  
  
可以发现 $S_i =1$ 是一个边界情况，这代表单调栈弹空，当前点是前缀最小值。  
  
那么从后往前，第一个 $1$ 最小，第二个 $1$ 次小，依次类推。这样可以使字典序最小。  
  
为什么一定是这样？因为正数第一个 $1$ 一定是当时的最小值，然后一旦出现更小的，势必会将栈弹空而得到 $1$。  
  
对于两个 $S = 1$ 中间的元素，一定都有：$p_i > p_L$，因此可以发现，$p_L$ 始终在栈中，直到遇到 $p_R$，而这段中倒数第一个 $S = 2$ 是最小……出现了，递归！  
  
那么对于确定的序列 $S$，我们已经可以解出 $P$ 了，然而有不确定元素。  
  
可以发现，对于不确定元素，$S_i = S_{i-1} + 1$，这样可以让大的数尽量在后面，保证字典序最小。  
  
然后打了个奇怪的 $O(n^2)$ 的垃圾实现，结果过了。。。。  
  
```cpp  
#include <cstdio>  
#include <vector>  
#include <algorithm>  
#include <cctype>  
using namespace std;  
const int bufSize = 1e6;  
inline char nc()  
{  
	return getchar();  
	static char buf[bufSize],*p1 = buf,*p2=buf;  
	return p1==p2&&(p2=(p1=buf)+fread(buf,1,bufSize,stdin),p1==p2)?EOF:*p1++;  
}  
template<typename T>  
inline T read(T &r)  
{  
	static char c;  
	static int flag;  
	r = 0,flag = 1;  
	for(c=nc();!isdigit(c);c=nc()) if(c == '-') flag = -1;  
	for(;isdigit(c);c=nc()) r=r*10+c-48;  
	return r*=flag;  
}  
const int maxn = 1e6 + 100;  
int n,S[maxn],a[maxn],minval;  
void solve(int l,int r)  
{  
	if(l > r) return;  
	vector<int> V;  
	V.push_back(r + 1);  
	for(int i = r;i>=l;--i) if(S[i] == 1) V.push_back(i),a[i] = ++minval;  
	V.push_back(l - 1);  
	for(int i = l;i<=r;++i) --S[i];  
	for(int i = V.size() - 1;i>0;--i) if(V[i] + 1 <= V[i-1] - 1) solve(V[i] + 1,V[i-1] - 1);  
}  
int main()  
{  
	read(n);  
	for(int i = 1;i<=n;++i) read(S[i]);  
	for(int i = 1;i<=n;++i) if(S[i] == -1) S[i] = S[i-1] + 1;  
	solve(1,n);  
	for(int i = 1;i<=n;++i) printf("%d ",a[i]);  
	return 0;  
}  
```  
  
要改成真的，其实也不难，记录递归层数，用桶排序去找当前为 $1$ 的点即可。  
  
然而这个货真价实的稳定 $O(n)$ 跑得还没能被卡到 $O(n^2)$ 的乱搞快。  
  
```cpp  
#include <cstdio>  
#include <vector>  
#include <algorithm>  
#include <cctype>  
using namespace std;  
const int bufSize = 1e6;  
inline char nc()  
{  
	return getchar();  
	static char buf[bufSize],*p1 = buf,*p2=buf;  
	return p1==p2&&(p2=(p1=buf)+fread(buf,1,bufSize,stdin),p1==p2)?EOF:*p1++;  
}  
template<typename T>  
inline T read(T &r)  
{  
	static char c;  
	static int flag;  
	r = 0,flag = 1;  
	for(c=nc();!isdigit(c);c=nc()) if(c == '-') flag = -1;  
	for(;isdigit(c);c=nc()) r=r*10+c-48;  
	return r*=flag;  
}  
const int maxn = 1e6 + 100;  
int n,S[maxn],a[maxn],head[maxn],minval;  
vector<int> V[maxn];  
void solve(int l,int r,int dep)  
{  
	if(l > r) return;  
	int tail = head[dep];  
	while(V[dep][tail + 1] <= r) ++tail;  
	for(int i = tail;i>=head[dep];--i) a[V[dep][i]] = ++minval;  
	if(l <= V[dep][head[dep]] - 1) solve(l,V[dep][head[dep]] - 1,dep + 1);  
	for(int i = head[dep] + 1;i<=tail;++i)   
		if(V[dep][i-1] + 1 <= V[dep][i] - 1)   
			solve(V[dep][i-1] + 1,V[dep][i] - 1,dep + 1);  
	if(V[dep][tail] + 1 <= r) solve(V[dep][tail] + 1,r,dep + 1);  
	head[dep] = tail + 1;  
}  
int main()  
{  
	read(n);  
	for(int i = 1;i<=n;++i) read(S[i]);  
	for(int i = 1;i<=n;++i) if(S[i] == -1) S[i] = S[i-1] + 1;  
	for(int i = 1;i<=n;++i) V[i].push_back(0),head[i] = 1;  
	for(int i = 1;i<=n;++i) V[S[i]].push_back(i);  
	for(int i = 1;i<=n;++i) V[i].push_back(n + 1);  
	solve(1,n,1);  
	for(int i = 1;i<=n;++i) printf("%d ",a[i]);  
	return 0;  
}  
```  
  
### P2605 [ZJOI2010]基站选址  
  
看起来很动态规划的题目。  
  
每个基站会影响前后的费用，可以先不考虑对后方的影响，有状态定义： $f(j,i)$ 为建立了 $j$ 个基站，且最后一个基站在 $i$ 点的当前最小费用。  
  
$$  f(j,i) = f(j-1,k) + C_i + cost(k + 1,i - 1)  $$  
  
其中 $cost(k + 1,i - 1)$ 代表区间 $[k+1,i-1]$ 在当前覆盖情况下的补偿费用。  
  
这个东西怎么计算？朴素计算方法就是大力遍历，$O(n)$ 查看。那么就能得到一个优秀的 $O(n^3k)$ 做法。  
  
如果在枚举过程中计算，就可以得到 $O(n^2k)$ 的优秀做法。  
  
考虑一个点 $i$ 会在什么时候产生补偿费用，可以二分出 $[L_i,R_i]$，代表如果有基站建造在 $[L_i,R_i]$ 中就不产生费用。反过来，$[1,L_i - 1]$ 与 $[R_i + 1,n]$ 就会产生费用。  
  
考虑建立一棵下标线段树，每个点代表 $f(j-1,k) + cost(k + 1,i - 1)$，每次动态更新 $cost(k+1,i-1)$，可以发现是个区间加法。  
  
然而这样会出现问题：未考虑到当前基站 $i$ 可以减少的费用。 因此我们需要妥善考虑何时将一个点加入费用线段树中。  
  
加入的点需要满足：在之前未被记费用、接下来会被记费用，可以发现，满足条件的是 $R_j = i$ 的点，因此可以把这些点对应存放，遍历时加入。  
  
维护一棵区间加、单点改、维护最小值的线段树。  
  
统计答案是个问题，考虑建立虚点，免费在本点建立基站，但距离要无穷大。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <vector>  
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
const int maxn = 21000;  
const long long inf = 1ll << 60;  
int n, k, dis[maxn], cost[maxn], range[maxn], L[maxn], R[maxn], w[maxn];  
long long sum[maxn], minn[maxn << 2], tag[maxn << 2];  
#define ls p << 1  
#define rs p << 1 | 1  
inline void pushdown(int p)  
{  
    if (!tag[p]) return;  
    minn[ls] += tag[p], minn[rs] += tag[p], tag[ls] += tag[p], tag[rs] += tag[p], tag[p] = 0;  
}  
inline void pushup(int p) { minn[p] = min(minn[ls], minn[rs]); }  
void modify(int l,int r,int p,int pos,long long k)  
{  
    if (l == r) return (void)(minn[p] = k);  
    int mid = (l + r) >> 1;  
    pushdown(p);  
    if (pos <= mid) modify(l, mid, ls, pos, k); else modify(mid + 1, r, rs, pos, k);  
    pushup(p);  
}  
void modify(int l, int r, int p, int ll, int rr, long long k)  
{  
    if (l >= ll && r <= rr) return (void)(minn[p] += k, tag[p] += k);  
    int mid = (l + r) >> 1;  
    pushdown(p);  
    if (ll <= mid) modify(l, mid, ls, ll, rr, k);  
    if (rr > mid) modify(mid + 1, r, rs, ll, rr, k);  
    pushup(p);  
}  
void build(int l, int r, int p)  
{  
    tag[p] = 0, minn[p] = 1 << 30;  
    if (l == r) return;  
    int mid = (l + r) >> 1;  
    build(l, mid, ls), build(mid + 1, r, rs);  
}  
vector<int> V[maxn];  
long long f[2][maxn];  
int main()  
{  
    read(n), read(k);  
    for (int i = 2; i <= n; ++i) read(dis[i]);  
    for (int i = 1; i <= n; ++i) read(cost[i]);  
    for (int i = 1; i <= n; ++i) read(range[i]);  
    for (int i = 1; i <= n; ++i) read(w[i]), sum[i] = sum[i - 1] + w[i];  
    for (int i = 1; i <= n; ++i)  
    {  
        int l = 1, r = i, mid, ans = i;  
        while (l <= r)  
        {  
            mid = (l + r) >> 1;  
            if (dis[mid] >= dis[i] - range[i]) ans = mid, r = mid - 1; else l = mid + 1;  
        }  
        L[i] = ans;  
        l = i, r = n, ans = i;  
        while (l <= r)  
        {  
            mid = (l + r) >> 1;  
            if (dis[mid] <= dis[i] + range[i]) ans = mid, l = mid + 1; else r = mid - 1;  
        }  
        R[i] = ans;  
    }  
    ++n, L[n] = R[n] = n, cost[n] = 0, w[n] = 1 << 30;  
    for (int i = 1; i <= n; ++i) V[R[i]].push_back(i);  
    long long res = inf;  
    int now = 1, last = 0;  
    {  
        long long tmp = 0;  
        for (int i = 1; i <= n; ++i)  
        {  
            f[last][i] = tmp + cost[i];  
            for (vector<int>::iterator it = V[i].begin(); it != V[i].end(); ++it) tmp += w[*it];  
        }  
        res = f[last][n];  
    }  
   for (int t = 2; t <= k + 1; ++t)  
    {  
        build(1, n, 1);  
        for (int i = 1; i <= n; ++i)  
        {  
            f[now][i] = minn[1] + cost[i];  
            for (vector<int>::iterator it = V[i].begin(); it != V[i].end(); ++it)  
                if (L[*it] > 1) modify(1, n, 1, 1, L[*it] - 1, w[*it]);  
            modify(1, n, 1, i, f[last][i]);  
        }  
        res = min(res, f[now][n]);  
        now ^= 1, last ^= 1;  
    }  
    printf("%lld\n", res);  
    return 0;  
}  
```  
  
### P2120 [ZJOI2007]仓库建设  
  
动态规划，定义 $f(i)$ 为最后一个仓库建立在 $i$ 点，只考虑前 $i$ 个工厂的最小费用。  
  
枚举上一个仓库进行转移。  
  
$$  f(i) = c_i + f(j) + \sum \limits _{k=j+1}^{i-1} p_k \times (x_i - x_k)  $$  
  
稍微整理一下式子。  
  
$$  f(i) = c_i + f(j) + x_i \times (\sum \limits _{k=j+1}^{i-1} p_k) - \sum \limits _{k=j+1}^{i-1} (p_k \times  x_k)  $$  
  
预处理前缀和 $S(i) = \sum \limits _{j=1}^i p_j$，$T(i) = \sum \limits _{j=1}^i p_j \times x_j$，就可以做到 $O(n^2)$ 转移：  
  
$$  f(i) = c_i + f(j) + x_i \times (S(i-1) - S(j)) - (T(i-1) - T(j))  $$  
  
只有一项与 $i$，$j$  同时有关，并且以乘积形式出现，考虑斜率优化。  
  
$$  x_i \times S(j) + f(i) - c_i - x_i \times S(i-1) + T(i-1) = f(j) + T(j)  $$  
  
令纵坐标$y = f(j) + T(j)$，横坐标 $x = S(j)$，截距 $f(i) - x_i \times S(i-1) + T(i-1) - c_i$，斜率 $k = x_i$，要求最小化截距。  
  
维护斜率递增的下凸包即可。  
  
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
const int maxn = 1e6 + 100;  
int n, dis[maxn], num[maxn], cost[maxn];  
int q[maxn], qt, qh;  
long long S[maxn], T[maxn], f[maxn];  
inline long long getx(int x) { return S[x]; }  
inline long long gety(int x) { return f[x] + T[x]; }  
inline bool cmp(int a, int b, int c) { return (gety(b) - gety(a)) * (getx(c) - getx(b)) >= (gety(c) - gety(b)) * (getx(b) - getx(a)); }  
inline bool cmp2(int a, int b, long long k)  
{  
    return gety(b) - gety(a) <= k * (getx(b) - getx(a));  
}  
int main()  
{  
    read(n);  
    for (int i = 1; i <= n; ++i) read(dis[i]),read(num[i]),read(cost[i]);  
    for (int i = 1; i <= n; ++i) S[i] = num[i] + S[i - 1], T[i] = 1ll * num[i] * dis[i] + T[i - 1];  
    q[++qt] = 0;  
    for (int i = 1; i <= n; ++i)  
    {  
        while(qt > qh && cmp2(q[qh],q[qh + 1],dis[i])) ++qh;  
        f[i] = cost[i] + f[q[qh]] + dis[i] * (S[i - 1] - S[q[qh]]) - (T[i - 1] - T[q[qh]]);  
        while (qt > qh && cmp(q[qt - 1], q[qt], i)) --qt;  
        q[++qt] = i;  
    }  
//    for(int i = 1;i<=n;++i) printf("%d %lld\n",i,f[i]);  
    printf("%lld\n", f[n]);  
    return 0;  
}  
```  
  
### [Codeforces Round #683 (Div. 2, by Meet IT)](https://codeforces.com/contest/1447)  
  
板刷该场比赛。  
  
#### A  
  
一开始有很多个元素，可以选择 $m$ 次操作，第 $i$ 次操作给除了你选一个元素外其他元素都加 $i$，其实就相当于给你选择的元素减去 $i$，构造一个方案让所有元素相等。  
  
然后发现初始时第 $i$ 个元素大小刚好是 $i$，于是只要顺次操作即可。  
  
```cpp  
#include <cstdio>  
int T,n;  
int main()  
{  
	scanf("%d",&T);  
	while(T--)  
	{  
		scanf("%d",&n);  
		printf("%d\n",n);  
		for(int i = 1;i<=n;++i) printf("%d ",i);  
		puts("");  
	}  
	return 0;  
}  
```  
  
#### B  
  
给一个矩阵，可以选择给相邻的两个元素同时取反。最大化所有元素和。  
  
每个元素可以被取反多次。  
  
感觉只要有两个负数，那么两个负数就可以被同时取反。利用中间值一步步传递过去。  
  
那么最后最多剩下一个负数，取绝对值最小的。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
const int maxn = 20;  
int T,n,m,a[maxn][maxn];  
int main()  
{  
	scanf("%d",&T);  
	while(T--)  
	{  
		scanf("%d %d",&n,&m);  
		for(int i = 1;i<=n;++i) for(int j = 1;j<=m;++j) scanf("%d",a[i] + j);  
		int num = 0,minn = 1 << 30;  
		for(int i = 1;i<=n;++i) for(int j = 1;j<=m;++j) num += (a[i][j] < 0),minn = std::min(minn,std::abs(a[i][j]));  
		int sum = 0;  
		for(int i = 1;i<=n;++i) for(int j = 1;j<=m;++j) sum += std::abs(a[i][j]);  
		if(num & 1) printf("%d\n",sum - 2 * minn);  
		else printf("%d\n",sum);  
	}  
	return 0;  
}  
```  
  
#### C  
  
好 ad-hocs 的题啊……  
  
首先，如果有任何一个 $w_i \ge \lceil \dfrac{W}{2} \rceil$，直接选择。扔掉所有比背包容量大的编号。  
  
剩下情况，$\forall w_i < \lceil \dfrac{W}{2}\rceil$.  
  
最少需要两件物品才行，可以考虑一路放过去，直到爆仓或者达到目标。  
  
什么时候会爆仓？原本满足 $\sum w_i < \lceil \dfrac{W}{2}\rceil$，放一件之后爆仓，可以发现是不可能的！  
  
所以只需要一路扫过去就可以了。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
const int maxn = 2e5 + 100;  
int T,n,a[maxn],s[maxn],top;  
long long W,mid;  
int main()  
{  
	scanf("%d",&T);  
	while(T--)  
	{  
		scanf("%d %lld",&n,&W);  
		top = 0;  
		for(int i = 1;i<=n;++i) scanf("%d",a + i);  
		mid = (W + 1) / 2;  
		long long now = 0;  
		for(int i = 1;i<=n;++i) if(a[i] <= W && a[i] >= mid){printf("1\n%d\n",i);goto end;}  
		for(int i = 1;i<=n;++i)  
		{  
			if(a[i] > W) continue;  
			s[++top] = i, now += a[i];  
			if(now >= mid)  
			{  
				printf("%d\n",top);  
				while(top) printf("%d ",s[top--]);  
				puts("");  
				goto end;  
			}  
		}  
		puts("-1");  
		end:;  
	}  
	return 0;  
}  
```  
  
#### D  
  
这种鬼畜组合题？  
  
首先回忆一下 最长公共子序列怎么求，定义 $f(i,j)$ 为 A 串匹配到前 $i$ 个字符，B 串匹配到前 $j$ 个字符时，最长公共子序列的长度。  
  
转移有：  
  
$$  f(i,j) = f(i-1,j-1) + 2, \text{ if } A_i = B_j  $$  
  
$$  f(i,j) = \max(f(i-1,j),f(i,j-1)), \text{ otherwise}  $$  
  
在本题中，重新定义 $f(i,j)$ 为 A 串中选出的子串最后一个字符为 $i$，B 串中选出的子串最后一个字符为 $j$ 时，最大的贡献。  
  
当 $A_i = B_j$ 时，可以发现有：  
  
$$  f(i,j) = f(i-1,j-1) + 2  $$  
  
如果前面已经有字符，那么这个转移方程就是正确的，因为子串必定连续。但有一个问题：可以从空串中转移。  
  
手动加上：  
  
$$  f(i,j) = \max(f(i-1,j-1),0) + 2  $$  
  
这就是个完备的式子了。  
  
其余情况：  
  
$$  f(i,j) = \max(f(i-1,j),f(i,j-1),0) - 1  $$  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
const int maxn = 5100;  
char A[maxn],B[maxn];  
int n,m,f[maxn][maxn];  
int main()  
{  
	scanf("%d %d",&n,&m);  
	scanf("%s\n%s",A + 1,B + 1);  
	int res = 0;  
	for(int i = 1;i<=n;++i) for(int j = 1;j<=m;++j)  
	{  
		if(A[i] == B[j]) f[i][j] = f[i-1][j-1] + 2;  
		f[i][j] = max(f[i][j],max(0,max(f[i-1][j],f[i][j-1])) - 1);  
		res = max(res,f[i][j]);  
	}  
	printf("%d\n",res);  
	return 0;  
}  
```  
  
#### E  
  
可以发现，每个点向外连一条边，一共有 $n$ 条无向边，而树只有 $n-1$ 条边，因此一定出现重边或者环。  
  
什么时候会出现重边？$a_x \oplus a_y$ 对 $a_x$ 与 $a_y$ 都是最小的时。  
  
一棵树中，一定只会有一条重边。首先，一棵树一定只有一条边多余，现在证明这条边一定是重边：考虑 $\min a_x \oplus a_y$，在所有异或和中，该对最小，那么对 $a_x$ 与 $a_y$ 一定都最小，即出现重边。  
  
原图被分割为若干个联通块，每个联通块都是带有重边的树。  
  
要求选出一些点，使整个图联通，即只存在一条重边。  
  
现在拆位考虑，从高到低，对于每位，按 $0/1$ 分类为集合 $S_0$，$S_1$，可以发现每个集合的点都只会向同集合内点连边。  
  
那么只要保留两个集合内部连边，就一定让原图分裂成两个联通块。  
  
因此需要将某个集合大小删到 $1$ 才能保证图联通。  
  
递归求解。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <vector>  
using namespace std;  
const int maxn = 2e5 + 100;  
int n,a[maxn];  
int solve(vector<int> &v,int bit)  
{  
	if(v.size() <= 3) return 0;  
	vector<int> S0,S1;  
	for(vector<int>::iterator it = v.begin();it!=v.end();++it) if((*it >> bit) & 1) S1.push_back(*it); else S0.push_back(*it);  
	vector<int>().swap(v);  
	if(S0.size() <= 1) return solve(S1,bit - 1);  
	if(S1.size() <= 1) return solve(S0,bit - 1);  
	int s0 = (int)(S0.size()) - 1,s1 = (int)(S1.size()) - 1;  
	return min(solve(S1,bit - 1) + s0,solve(S0,bit - 1) + s1);  
}  
int main()  
{  
	scanf("%d",&n);  
	vector<int> v;  
	for(int i = 1,x;i<=n;++i) scanf("%d",&x),v.push_back(x);  
	printf("%d\n",solve(v,30));  
	return 0;  
}  
```  
  
#### F1  
  
出现次数最多的元素有多个。  
  
查看整个数组出现次数最多的元素，将它的出现次数减少到与次多的一致。  
  
然而可能会在减少的过程中影响到次多的元素的个数。  
  
由于值域较小，可以直接枚举次多的元素是哪个，然后遍历整个数组去检查。  
  
做个只关于这两个元素的前缀和，令 $sum1(r) - sum1(l-1) = sum2(r) - sum2(l-1)$ 即可，调整下形式可以得到 $sum1(r) - sum2(r) = sum1(l - 1) - sum2(l - 1)$，开个桶记录。  
  
那么可以得出相应的区间，使原本出现次数最多的元素与当前钦定的元素出现次数相等。但这有一个问题：它们可能不再是出现次数最多的元素了。  
  
不过可以发现，如果有元素在对应区间出现次数更多，那么在枚举到那个元素时，能计算到的区间长度就更大，最后答案是正确的。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
const int maxn = 2e5 + 100;  
int n,a[maxn],cnt[110];  
int vis[maxn * 2],last[maxn * 2],tim;  
int main()  
{  
	scanf("%d",&n);  
	for(int i = 1;i<=n;++i) scanf("%d",a + i),cnt[a[i]]++;  
	int maxx = 1;  
	for(int i = 1;i<=100;++i) if(cnt[i] > cnt[maxx]) maxx = i;  
	int res = 0;  
	for(int t = 1;t<=100;++t)  
	{  
		if(t == maxx) continue;  
		int now = 0;  
		++tim;  
		vis[now + 200000] = tim,last[now + 200000] = 0;  
		for(int i = 1;i<=n;++i)  
		{  
			if(a[i] == maxx) ++now;  
			else if(a[i] == t) --now;  
			if(vis[now + 200000] == tim) res = max(res,i - last[now + 200000]);  
			else vis[now + 200000] = tim,last[now + 200000] = i;  
		}  
	}  
	printf("%d\n",res);  
	return 0;  
}  
```  
  
#### F2  
  
值域变大了，上一题的枚举就做不了了。  
  
考虑枚举左端点？也没法做。  
  
看看题解，根号做法？  
  
所有数的出现次数的和一定，那么出现次数大于 $\sqrt{n}$ 的数至多有 $\sqrt{n}$ 个。  
  
这部分可以大力枚举解决。跟 F1 一样的做法。  
  
然后考虑枚举出现次数。  
  
出现次数只有 $\sqrt{n}$ 个，此时枚举的是答案的次数，满足条件需要：出现次数 $i$ 出现的次数 $\ge 2$，可以用类似莫队的技巧去维护。  
  
双指针扫一遍。  
  
时间复杂度  $O(n\sqrt{n})$.  
  
```cpp  
#include <cstdio>  
#include <cmath>  
#include <algorithm>  
using namespace std;  
const int maxn = 2e5 + 100;  
int n,a[maxn],times[maxn],B,s[maxn],top,vis[maxn * 2],tim,last[maxn * 2],res,cnt[maxn],tot[maxn];  
inline void add(int x) { tot[cnt[a[x]]++]--,tot[cnt[a[x]]]++; }  
inline void del(int x) { tot[cnt[a[x]]--]--,tot[cnt[a[x]]]++; }  
int main()  
{  
	scanf("%d",&n);  
	for(int i = 1;i<=n;++i) scanf("%d",a + i),times[a[i]]++;  
	int maxx = 1;  
	for(int i = 2;i<=n;++i) if(times[i] > times[maxx]) maxx = i;  
	B = std::sqrt(n);  
	for(int i = 1;i<=n;++i) if(i!=maxx && times[i] > B) s[++top] = i;  
	for(int t = 1,now;t<=top;++t)  
	{  
		now = 0,vis[200000] = ++tim,last[200000] = 0;  
		for(int i = 1;i<=n;++i)   
		{  
			if(a[i] == maxx) ++now; else if(a[i] == s[t]) --now;  
			if(vis[now + 200000] == tim) res = max(res,i - last[now + 200000]); else vis[now+200000] = tim,last[now + 200000] = i;  
		}  
	}  
	for(int t = 1;t<=B;++t)  
	{  
		for(int i = 1;i<=n;++i) tot[i] = cnt[i] = 0;  
		int l = 1,r = 0;  
		while(r < n)  
		{  
			add(++r);  
			while(cnt[a[r]] > t) del(l++);  
			if(tot[t] >= 2) res = max(res,r - l + 1);  
		}  
	}  
	printf("%d\n",res);  
	return 0;  
}  
```  
  
### [Codeforces Round #684 (Div. 2)](https://codeforces.com/contest/1440)  
  
#### A  
  
简单的贪心，要么原串要么全零，要么全一。  
  
懒得讨论了，直接都算一遍取个最小值。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
int T,n,c0,c1,h;  
char s[10000];  
int main()  
{  
	scanf("%d",&T);  
	while(T--)  
	{  
		scanf("%d %d %d %d %s",&n,&c0,&c1,&h,s + 1);  
		int num0 = 0,num1 = 0;  
		for(int i = 1;i<=n;++i) num0 += s[i] == '0',num1 += s[i] == '1';  
		printf("%d\n",min(num0 * c0 + num1 * c1,min(num0 * h + n * c1,num1 * h + n * c0)));  
	}  
	return 0;  
}  
```  
  
#### B  
  
排序一遍，然后从后往前数数取一下就行了。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
const int maxn = 2e5 + 100;  
int T,n,k,a[maxn];  
int main()  
{  
	scanf("%d",&T);  
	while(T--)  
	{  
		scanf("%d %d",&n,&k);  
		for(int i = 1;i<=n*k;++i) scanf("%d",a + i);  
		sort(a + 1,a + 1 + n * k);  
		int right = n - (n + 1) / 2;  
		long long sum = 0;  
		for(int i = 1;i<=k;++i) sum += a[n*k - (right+1) * (i-1) - right];  
		printf("%lld\n",sum);  
	}  
	return 0;  
}  
```  
  
#### C  
  
在一个 $2 \times 2$ 方格中考虑。  
  
如果有三个 $1$，那么一次操作解决。  
  
如果有两个 $1$，那么可以一次操作转化为三个一，然后一次操作解决。  
  
如果有一个 $1$，那么可以一次操作转化为两个 $1$.  
  
如果有四个 $1$，那么可以一次操作转化为一个 $1$.  
  
那么：三个需要一次操作，两个需要两次操作，一个需要三次操作，四个需要四次操作。  
  
刚好卡到上界。  
  
问题在于，可能不能正好分割为 $2 \times 2$ 的方格。  
  
可以把所有的 $1$ 从上到下、从左到右压到最右下角的一个 $2 \times 2$ 的方格里。  
  
具体地，对于平凡的点，如果为 $1$，那么将它的下方、右下方和它一起翻转。如果是最后一个，就将它的下方、左下方一起翻转。  
  
直到两行，可以从左到右一路翻转。  
  
这样每个点至多操作一次，最后在 $2 \times 2$ 的方格中乱搞一下即可。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
int T,n,m,a[110][110];  
int X[1000000],Y[1000000],cnt;  
char s[110];  
inline void add(int x,int y)  
{  
	X[++cnt] = x,Y[cnt] = y,a[x][y] ^= 1;  
}  
void solve(int n,int m)  
{  
	int num = a[n][m] + a[n-1][m] + a[n][m-1] + a[n-1][m-1];  
		if(num == 4)  
		{  
			add(n,m),add(n-1,m),add(n,m-1);  
			goto one;  
		}  
		else if(num == 3)  
		{  
			three:  
			if(a[n][m]) add(n,m);  
			if(a[n-1][m]) add(n-1,m);  
			if(a[n][m-1]) add(n,m-1);  
			if(a[n-1][m-1]) add(n-1,m-1);  
		}  
		else if(num == 2)  
		{  
			two:  
			bool aa = 0,b = 0,c = 0,d = 0;  
			if(!a[n][m]) add(n,m),aa = 1;  
			if(!a[n-1][m]) add(n-1,m),b = 1;  
			if(!a[n][m-1]) add(n,m-1),c = 1;  
			if(!a[n-1][m-1]) add(n-1,m-1),d = 1;  
			if(!aa && a[n][m]) add(n,m);  
			else if(!b && a[n-1][m]) add(n-1,m);  
			else if(!c && a[n][m-1]) add(n,m-1);  
			else if(!d && a[n-1][m-1]) add(n-1,m-1);  
			goto three;  
		}  
		else if(num == 1)  
		{  
			one:  
			if(a[n][m]) add(n,m),add(n-1,m),add(n,m-1);  
			else if(a[n-1][m]) add(n-1,m),add(n,m),add(n,m-1);  
			else if(a[n][m-1]) add(n,m-1),add(n,m),add(n-1,m-1);  
			else if(a[n-1][m-1]) add(n-1,m-1),add(n,m),add(n,m-1);  
			goto two;  
		}  
}  
int main()  
{  
	scanf("%d",&T);  
	while(T--)  
	{  
		scanf("%d %d",&n,&m);  
		cnt = 0;  
		for(int i = 1;i<=n;++i)  
		{  
			scanf("%s",s + 1);  
			for(int j = 1;j<=m;++j) a[i][j] = s[j] - '0';  
		}  
		for(int i = 1;i<= n - 2;++i) for(int j = 1;j<=m;++j)  
		{  
			if(a[i][j])   
			{  
				add(i,j);  
				add(i+1,j);  
				if(j == m) add(i+1,j-1);  
				else add(i+1,j+1);  
			}  
		}  
		for(int j = 1;j<=m-2;++j)   
		{  
			if(a[n-1][j] && a[n][j]) add(n-1,j),add(n,j),add(n-1,j + 1);  
			else if(a[n-1][j]) add(n-1,j),add(n-1,j+1),add(n,j+1);  
			else if(a[n][j]) add(n,j),add(n-1,j+1),add(n,j+1);  
		}  
  
		solve(n,m);  
		printf("%d\n",cnt / 3);  
		for(int i = 1;i<=cnt;++i)   
		{  
			printf("%d %d ",X[i],Y[i]);  
			if((i % 3) == 0) puts("");  
		}  
		//for(int i = 1;i<=n;puts(""),++i) for(int j = 1;j<=m;++j) printf("%d ",a[i][j]);  
	}  
	return 0;  
}  
```  
  
### [Educational Codeforces Round 98 (Rated for Div. 2)](https://codeforces.com/contest/1452)  
  
#### A  
  
斜着走很优。然后就可以两步走一格。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
int T,x,y;  
int main()  
{  
	scanf("%d",&T);  
	while(T--)  
	{  
		scanf("%d %d",&x,&y);  
		int minn = std::min(x,y),res = minn * 2;  
		x -= minn,y -= minn,x += y;  
		if(x == 0) {printf("%d\n",res);continue;}  
		printf("%d\n",res + 2 * x - 1);  
	}  
}  
```  
  
#### B  
  
假定调整后的总和为 $sum$，平均后的元素就是 $\dfrac{sum}{n-1}$，要求最大元素不超过这个，即 $maxx \le \dfrac{sum}{n-1}$，反过来就是要求 $sum \ge maxx \times (n - 1)$。  
  
找出原数组中最大元素，对应进行调整？  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
const int maxn = 2e5 + 100;  
int T,n,a[maxn];  
int main()  
{  
	scanf("%d",&T);  
	while(T--)  
	{  
		scanf("%d",&n);  
		int maxx = 0;  
		long long sum = 0,oldsum;  
		for(int i = 1;i<=n;++i) scanf("%d",a + i),maxx = max(maxx,a[i]),sum += a[i];  
		oldsum = sum;  
		sum = (sum + n - 2) / (n - 1) * (n - 1);//round up  
		sum = max(sum,1ll * maxx * (n - 1));  
		printf("%lld\n",sum - oldsum);  
	}  
	return 0;  
}  
```  
  
#### C  
  
每次删除一个子序列，要求子序列是合法括号串。  
  
要求最大化删除次数。  
  
考虑每次只删一个括号，能删则删，两种括号并不互相影响。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <cstring>  
using namespace std;  
const int maxn = 2e5 + 100;  
char s[maxn];  
int T,n,num1,num2;  
int main()  
{  
	scanf("%d",&T);  
	while(T--)  
	{  
		scanf("%s",s + 1);  
		n = strlen(s + 1);  
		int res = 0;  
		num1 = num2 = 0;  
		for(int i = 1;i<=n;++i)  
		{  
			if(s[i] == '(') ++num1;  
			else if(s[i] == ')')  
			{  
				if(num1) ++res,--num1;  
			}  
			else if(s[i] == '[') ++num2;  
			else  
			{  
				if(num2) ++res,--num2;  
			}  
		}  
		printf("%d\n",res);  
	}  
	return 0;  
}  
```  
  
#### D  
  
考虑第一个塔在哪里，覆盖范围是确定的，然后就分治成了一个子问题？  
  
现在需要覆盖 $[L,R]$ 间的所有位置，枚举第一座塔 $i$，然后就分治成了覆盖 $[2 \times i - L,R]$.  
  
然后发现右端点是永远不变的，因此可以倒着做一个动态规划。  
  
$f(i)$ 代表覆盖 $[i,R]$ 间的概率。枚举第一座塔进行转移。  
  
$$  
f(i) = \sum \limits _{k=0} ^{i + 2 \times k + 1 \le n} \dfrac{1}{2^{k+1}} \times f(i + 2 \times k + 1)  
$$  
  
稍微改写一下：  
  
$$  f(i) = 2^i \times \sum \limits _{k=0} ^{i + 2 \times k + 1 \le n} \dfrac{1}{2^{i + k+1}} \times f(i + 2 \times k + 1)  $$  
  
分奇偶记录前缀和即可。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
const int mod = 998244353;  
const int inv = 499122177;//inv(2)  
inline int add(int x,int y){int res = x + y;return res >= mod ? res - mod : res;}  
inline int mul(int x,int y)  
{  
	long long res = 1ll * x * y;  
	return res >= mod ? res % mod : res;  
}  
const int maxn = 2e5 + 100;  
int n,f[maxn],g[maxn][2],prod[maxn],prodinv[maxn];  
int main()  
{  
	scanf("%d",&n);  
	prod[0] = prodinv[0] = 1;  
	for(int i = 1;i<=n + 1;++i) prod[i] = mul(prod[i-1],2),prodinv[i] = mul(prodinv[i-1],inv);  
	f[n + 1] = 1,g[n + 1][(n + 1) & 1] = prodinv[n+1];  
	for(int i = n;i;--i)  
	{  
		f[i] = mul(prod[i],g[i+1][(i+1) & 1]);  
		g[i][0] = g[i+1][0],g[i][1] = g[i+1][1];  
		g[i][i & 1] = add(g[i][i & 1],mul(prodinv[i],f[i]));  
	}  
	printf("%d\n",f[1]);  
	return 0;  
}  
```  
  
看了看题解，发现其实只要将 $[1,n]$ 划分为若干个奇数段即可。  
  
设 $f(i)$  为划分完 $[1,i]$ 的方案数，有：  
  
$$  f(i) = f(i-1) + f(i-2)  $$  
  
即考虑最后一位可以单独一段，也可以后两位接到最后一段上。  
  
这个转移可以矩阵加速。  
  
#### E  
  
对于每个学生的区间 $[L,R]$，有中心点 $mid = \dfrac{L+R}{2}$，使讲师区间越靠近中心点，对学生吸引力越大。  
  
按照 $mid$ 对学生排序，即可保证在某点处，前一部分学生都倾向一个讲师，后一部分学生都倾向另一个讲师。  
  
可以考虑枚举这个分界点，那么只需要计算对于前一部分学生、后一部分学生，放置长度为 $k$ 的讲师区间能得到的最大贡献值。  
  
$O(nm)$ 枚举学生，然后枚举对应位置即可预处理。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
const int maxn = 2100;  
int n,m,k,prefix[maxn],suffix[maxn],pos[maxn];  
struct node  
{  
	int l,r;  
}A[maxn];  
bool cmp(const node &a,const node &b)  
{  
	return a.l + a.r < b.l + b.r;  
}  
int main()  
{  
	scanf("%d %d %d",&n,&m,&k);  
	for(int i = 1;i<=m;++i) scanf("%d %d",&A[i].l,&A[i].r);  
	if(m == 1) return printf("%d\n",min(k,A[1].r - A[1].l + 1)),0;  
	sort(A + 1,A + 1 + m,cmp);  
	for(int i = 1; i <= m; ++i)  
	{  
		for(int j = 1; j + k - 1 <= n; ++j)  
		{  
			int l = max(A[i].l,j),r = min(A[i].r,j + k - 1);  
			if(l <= r) pos[j] += r - l + 1;  
			prefix[i] = max(prefix[i],pos[j]);  
		}  
	}  
	for(int i = 1;i<=n;++i) pos[i] = 0;  
	for(int i = m;i;--i)  
	{  
		for(int j = 1;j + k - 1 <= n;++j)  
		{  
			int l = max(A[i].l,j),r = min(A[i].r,j + k - 1);  
			if(l <= r) pos[j] += r - l + 1;  
			suffix[i] = max(suffix[i],pos[j]);  
		}  
	}  
	int res = 0;  
	for(int i = 1;i<m;++i) res = max(res,prefix[i] + suffix[i + 1]);  
	printf("%d\n",res);  
	return 0;  
}  
```  
  
### [Codeforces Round #687 (Div. 2, based on Technocup 2021 Elimination Round 2)](https://codeforces.com/contest/1457)  
  
#### A  
  
曼哈顿距离最大值？  
  
四个角。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
int T,n,m,r,c;  
int main()  
{  
	scanf("%d",&T);  
	while(T--)  
	{  
		scanf("%d %d %d %d",&n,&m,&r,&c);  
		printf("%d\n",max(r - 1 + c - 1,max(n - r + c - 1,max(r - 1 + m - c,n - r + m - c))));  
	}  
	return 0;  
}  
```  
  
#### B  
  
枚举一个最终颜色，从左到右遍历，遇到一个不同就刷，然后指针直接向后跳。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
const int maxn = 1e5 + 100;  
int T,n,k,a[maxn];  
int main()  
{  
	scanf("%d",&T);  
	while(T--)  
	{  
		scanf("%d %d",&n,&k);  
		int maxx = 0;  
		for(int i = 1;i<=n;++i) scanf("%d",a + i),maxx = max(maxx,a[i]);  
		int res = 1 << 30;  
		for(int c = 1;c<=maxx;++c)  
		{  
			int now = 0;  
			for(int i = 1;i<=n;++i)  
				if(a[i] != c) ++now,i += k - 1;  
			res = min(res,now);  
		}  
		printf("%d\n",res);  
	}  
	return 0;  
}  
```  
  
#### C  
  
相对操作，在开头删除一个，相当于出发点向后移一位。  
  
维护对于对应出发点的贡献即可，枚举出发点统计答案。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
const int maxn = 1e5 + 100;  
int T,n,p,k,cnt[maxn],x,y;  
char a[maxn];  
int main()  
{  
	scanf("%d",&T);  
	while(T--)  
	{  
		scanf("%d%d%d",&n,&p,&k);  
		scanf("%s",a + 1);  
		scanf("%d %d",&x,&y);  
		for(int i = 0;i<k;++i) cnt[i] = 0;  
		int res = 1 << 30;  
		for(int i = n;i >= p;--i)  
		{  
			cnt[i % k] += (a[i] == '0') * x;  
			res = min(res,y * (i - p) + cnt[i % k]);  
		}  
		printf("%d\n",res);  
	}  
	return 0;  
}  
```  
  
#### D  
  
如果所有数的最高位都不相同，那么肯定无解。因为异或之后最高位不变，那么相对大小永远不变。  
  
如果有超过两个连续的最高位相同，那么一步就有解。  
  
感觉这个多个连续最高位相同的可能性很大，因为如果都要求不同，最多只有三十种最高位，那么数组最多就有六十的长度。  
  
这个数据范围考虑乱搞。  
  
最终有一个 $a_i > a_{i+1}$，可以发现这两个东西一定是两端区间异或和的结果，所以可以考虑直接枚举分界点，然后枚举左右端点，即使 $O(n^3)$ 的复杂度也无所谓。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
const int maxn = 1e5 + 100;  
int n,a[maxn],hb[maxn],sum[maxn];  
int main()  
{  
	scanf("%d",&n);  
	for(int i = 1;i<=n;++i) scanf("%d",a + i);  
	for(int i = 1;i<=n;++i) for(int j = 30;j>=0;--j) if((a[i] >> j) & 1) { hb[i] = j; break; }  
	for(int i = 1;i+2<=n;++i) if(hb[i] == hb[i+1] && hb[i+1] == hb[i+2]) {puts("1");return 0;}  
	for(int i = 1;i<=n;++i) sum[i] = sum[i-1] ^ a[i];  
	int res = 1 << 30;  
	for(int i = 1;i<n;++i)  
		for(int l = i;l;--l)  
			for(int r = i + 1;r<=n;++r)  
				if((sum[i] ^ sum[l-1]) > (sum[r] ^ sum[i]))  
					res = min(res,i - l + r - i - 1);  
	if(res > n) puts("-1");  
	else printf("%d\n",res);  
	return 0;  
}  
```  
  
#### E  
  
首先，一定不会在当前分数为正时清零。  
  
然后，根据套路，先打贡献分数大的。  
  
考虑什么时候清空。这个显然可以有 $O(nk)$ 的动态规划来做，只是时空双炸，不可接受。  
  
口胡了一下没想到怎么优化，看了官方题解，写得不错。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <queue>  
using namespace std;  
const int maxn = 5e5 + 100;  
int n,k,a[maxn];  
priority_queue<long long> q;  
int main()  
{  
	scanf("%d %d",&n,&k);  
	for(int i = 1;i<=n;++i) scanf("%d",a + i);  
	sort(a + 1,a + 1 + n);  
	for(int i = 1;i<=k + 1;++i) q.push(0ll);  
	long long res = 0;  
	for(int i = n;i;--i)  
	{  
		long long t = q.top();  
		q.pop(),res += t,q.push(t + a[i]);  
	}  
	printf("%lld\n",res);  
	return 0;  
}  
```  
  
#### F  
  
官方题解写得好。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
const int maxn = 5100;  
#define int long long  
struct node  
{  
	int t,x;  
}A[maxn];  
inline bool cmp(const node &a,const node &b){return a.t < b.t;}  
int n,f[maxn][maxn],minn[maxn];  
inline int dis(int a,int b){return abs(A[a].x - A[b].x);}  
signed main()  
{  
	scanf("%lld",&n);  
	for(int i = 1;i<=n;++i) scanf("%lld %lld",&A[i].t,&A[i].x);  
	sort(A + 1,A + 1 + n,cmp);  
	for(int i = 1;i<=n;++i) minn[i] = 1 << 30;  
	f[0][0] = 1;  
	for(int i = 0;i<n;++i)  
	{  
		if(minn[i] <= A[i].t)  
		{  
			minn[i + 1] = min(minn[i + 1],max(A[i].t,minn[i] + dis(i,i+1)));  
			for(int j = i + 2;j<=n;++j)//place a clone  
				if(max(A[i].t, minn[i] + dis(i,j)) + dis(j,i+1) <= A[i+1].t) f[i+1][j] = 1;  
		}  
		if(i < n - 1 && f[i][i+1]) //go directly to i + 2 and leave the i + 1 to clone onw  
		{  
			minn[i+2] = min(minn[i+2],max(A[i+1].t,A[i].t + dis(i,i + 2)));  
			for(int j = i + 3;j<=n;++j) if(max(A[i+1].t,A[i].t + dis(i,j)) + dis(j,i + 2) <= A[i+2].t)  
				f[i+2][j] = 1;	  
		}  
		if(A[i].t + dis(i,i+1) <= A[i+1].t) for(int j = i + 2;j<=n;++j) f[i+1][j] |= f[i][j];  
	}  
	if(minn[n] <= A[n].t || f[n-1][n]) puts("YES"); else puts("NO");  
	return 0;  
}   
```  
  
### [Educational Codeforces Round 99 (Rated for Div. 2)](https://codeforces.com/contest/1455)  
  
#### A  
  
看起来跟位数有关？  
  
分母的含义就是去掉后导零。  
  
那么除非有后导零，否则值为 $1$，考虑有后导零时，结果与后导零有关。  
  
$g(x)$ 就是数 $x$ 的后导零数量再加一。  
  
那么数数位数就可以了。  
  
```cpp  
#include <cstdio>  
#include <cstring>  
using namespace std;  
int T;  
char s[110];  
int main()  
{  
	scanf("%d",&T);  
	while(T--)  
	{  
		scanf("%s",s + 1);  
		int n = strlen(s + 1);  
		printf("%d\n",n);  
	}  
	return 0;  
}  
```  
  
#### B  
  
可以往回跳再往前跳，在第 $i$ 步决策往回跳实质上减小了 $i + 1$ 步。  
  
考虑先跳过，再调整。$\dfrac{n\times(n+1)}{2} \ge x$，如果恰好等于就结束了，否则，可以发现上一步跳了 $n$ 步，那么 $x$ 与当前位置的距离一定 $< n$，那么一定可以选择一个 $i + 1 < n$ 来把最后多余的消掉。  
  
除非 $i=0$，特判一下。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <cmath>  
using namespace std;  
int T,x;  
int main()  
{  
	scanf("%d",&T);  
	while(T--)  
	{  
		scanf("%d",&x);  
		int t = sqrt(x * 2);  
		while(t * (t+1)/2 < x) ++t;  
		printf("%d\n",t + (t *(t+1)/2 == (x + 1)));  
	}  
	return 0;  
}  
```  
  
#### C  
  
一方出球，一方如果不回，直接被得分。而省下来的球最多得一分。弱势的一方一直不回，直到对方没力气了再打。另一方为了最大化得分，一定一直发球。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
int T,x,y;  
int main()  
{  
	scanf("%d",&T);  
	while(T--)  
	{  
		scanf("%d %d",&x,&y);  
		printf("%d %d\n",x - 1,y);  
	}  
}  
```  
  
#### D  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
int T,n,x,a[11000];  
int main()  
{  
	scanf("%d",&T);  
	while(T--)  
	{  
		scanf("%d %d",&n,&x);  
		for(int i = 1;i<=n;++i) scanf("%d",a + i);  
		int res = 0;  
		for(int i = 1;i<=n;++i) if(a[i] > x)   
		{  
			bool flag = 0;  
			for(int i = 1;i<n;++i) if(a[i] > a[i+1]) flag = 1;  
			if(!flag) break;  
			std::swap(a[i],x),++res;  
		}  
		for(int i = 1;i<n;++i) if(a[i] > a[i+1]) goto no;  
		printf("%d\n",res);continue;  
		no: puts("-1");  
	}  
	return 0;  
}  
```  
  
#### E  
  
目标正方形四角：$X,Y,X + D,Y + D$，全排列枚举四角，左下 $(x_1,y_1)$，左上 $(x_2,y_2)$，右下 $(x_3,y_3)$，右上 $(x_4,y_4)$，有曼哈顿距离和：  
  
$$  |x_1 - X| + |x_2 - X| + |x_3 - X - D| + |x_4 - X - D| + |y_1 - Y| + |y_3 - Y| + |y_2 - Y - D| + |y_4 - Y - D|  $$  
  
当 $X \in [\min(x_1,x_2),\max(x_1,x_2)]$ 时，$|x_1 - X| + |x_2 - X| = |x_1 - x_2|$ 最小。  
  
当 $X + D \in [\min(x_3,x_4),\max(x_3,x_4)]$ 时，$|x_3 - X - D| + |x_4 - X - D| = |x_3 - x_4|$ 最小。  
  
这两个都是变量，看起来很美好，都能取到，然而还要看 $Y + D$  的情况。  
  
当 $Y \in [\min(y_1,y_3),\max(y_1,y_3)]$ 时，$|y_1 - Y| + |y_3 - Y| = |y_1 - y_3|$ 最小。  
  
当 $Y + D \in [\min(y_2,y_4),\max(y_2,y_4)]$ 时，$|y_2 - Y - D| + |y_4 - Y - D| = |y_2 - y_4|$ 最小。  
  
这里面只有 $D$ 是会互相影响的。  
  
考虑当 $X$ 取最优时，对应的 $X + D$ 能取到最优的对应的 $D$ 是有范围的，求出这个范围。  
  
当 $Y$ 取最优时同理，可以发现得到了两个关于 $D$ 的线段，如果相交那么大家都最优。  
  
否则呢？可以发现，对于 $X + D$ 与 $Y + D$，每偏离最优区间一步，就会产生 $2$ 的额外代价。让 $D$ 在两个最优的 $D$ 区间线段中间的空白处，就可以最小化额外代价，即两倍的中间空白处的长度。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
#define int long long  
int T,x[5],y[5],p[5];  
signed main()  
{  
	scanf("%lld",&T);  
	while(T--)  
	{  
		for(int i = 1;i<=4;++i) scanf("%lld %lld",x + i,y + i),p[i] = i;  
		long long res =  1ll << 60;  
		do  
		{  
			int x1 = x[p[1]],x2 = x[p[2]],x3 = x[p[3]],x4 = x[p[4]];  
			int y1 = y[p[1]],y2 = y[p[2]],y3 = y[p[3]],y4 = y[p[4]];  
			long long now = abs(x1 - x2) + abs(x3 - x4) + abs(y1 - y3) + abs(y2 - y4);  
			int l1 = min(x3,x4) - max(x1,x2),r1 = max(x3,x4) - min(x1,x2);  
			int l2 = min(y2,y4) - max(y1,y3),r2 = max(y2,y4) - min(y1,y3);  
			if(l1 > l2) std::swap(l1,l2),std::swap(r1,r2);  
			if(r1 < l2) now += (l2 - r1) * 2;  
			res = min(res,now);  
		}while(next_permutation(p + 1,p + 5));  
		printf("%lld\n",res);  
	}  
	return 0;  
}  
```  
  
#### F  
  
要求字典序最小。可以发现，每个字符最多只被操作一次。  
  
一个字符基本没法到达别的位置，可以发现，第 $i$ 个字符最多到达 $i+2$ 的位置，也许可以想办法大力转移？  
  
定义 $f(i)$ 为前 $i$ 个字符操作完后可能的最小字典序结果串。  
  
现在考虑 $i+1$ 字符的操作，如果是单字符加减或不变那么直接有：  
  
$$  f(i+1) = f(i) + c'  $$  
  
难点在于交换。可以发现往左交换是比较 trivial 的，因为状态已经确定了。  
  
$$  f(i+1) = f(i) - c_i' + c_{i+1} + c_i'  $$  
  
大概就是这么个样子。  
  
如果往右交换呢？那决策点直接又变成了 $i + 2$ 字符，好在此时可以发现：这个字符不会再向右交换了。  
  
如果它再向右，就相当于交换两次没有交换，因此可以直接枚举选择暴力转移到 $f(i+2)$ 位置。  
  
大概可以做到 $O(n)$ 单次，因为有相当长的字符串是固定的，只需要记录后几个字符。但无所谓了， $O(n^2)$ 足矣。  
  
### Atcoder Regular Contest 109  
  
#### A  
  
有两栋楼，要从 $A$ 栋的 $a$ 层到 $B$ 栋的 $b$ 层，跨楼走一层代价为 $x$，同楼走一层代价为 $y$.  
  
直接乱搞个最短路就行了。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <queue>  
using namespace std;  
const int maxn = 1e4;  
struct node  
{  
	int to,next,val;  
}E[maxn];  
int head[maxn],tot;  
inline void add(int x,int y,int w){E[++tot].next = head[x],head[x] = tot,E[tot].to = y,E[tot].val = w;}  
int a,b,x,y;  
int dis[maxn],vis[maxn];  
int main()  
{  
	scanf("%d %d %d %d",&a,&b,&x,&y);  
	for(int i = 1;i<100;++i) add(i,i + 1,y),add(i + 1,i,y);  
	for(int i = 1;i<=100;++i) add(i,i + 100,x),add(i + 100,i,x);  
	for(int i = 1;i<100;++i) add(i + 1,i + 100,x),add(i + 100,i + 1,x);  
	priority_queue<pair<int,int>,vector<pair<int,int> >,greater<pair<int,int> > > q;  
	for(int i = 1;i<=200;++i) dis[i] =  1<<30;  
	dis[a] = 0;  
	q.push(make_pair(0,a));  
	while(q.size())  
	{  
		int u = q.top().second;  
		q.pop();  
		if(vis[u]) continue;  
		vis[u] = 1;  
		for(int p = head[u];p;p=E[p].next)  
		{  
			int v = E[p].to;  
			if(!vis[v] && dis[v] > dis[u] + E[p].val)  
			{  
				dis[v] = dis[u] + E[p].val,q.push(make_pair(dis[v],v));  
			}  
		}  
	}  
	printf("%d\n",dis[100 + b]);  
	return 0;  
}  
```  
  
#### B  
  
给 $n+1$ 的排列，可以将每个数任意分割，求最少的选取的元素个数，以获取 $n$ 的排列。  
  
也就是任意分配最后一个元素 $n + 1$ 罢了。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <cmath>  
using namespace std;  
long long n;  
int main()  
{  
	scanf("%lld",&n);  
	long long t = sqrt(2 * (n + 1));  
	while(t * (t + 1) / 2 <= (n + 1)) ++t;  
	--t;  
	printf("%lld\n",n - t + 1);  
	return 0;  
}  
```  
  
#### C  
  
可以考虑从字符串开始模拟。  
  
如果当前字符串是奇数，翻倍使其成为偶数。然后向上打一层。  
  
然后如果是奇数，继续翻倍。可以发现，每层向上最多减半再翻倍，总长不变。  
  
然后一共打 $k$ 次，取最上面一层的第一个字符作为胜者。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
#include <cstring>  
using namespace std;  
char f[200][200];  
int n,k;  
char s[100000],s2[100000];  
int main()  
{  
	f['R']['S'] = f['S']['R'] = 'R';  
	f['P']['R'] = f['R']['P'] = 'P';  
	f['S']['P'] = f['P']['S'] = 'S';  
	f['R']['R'] = 'R';  
	f['S']['S'] = 'S';  
	f['P']['P'] = 'P';  
	scanf("%d %d",&n,&k);  
	scanf("%s",s + 1);  
	for(int t = 1;t<=k;++t)  
	{  
		if(n & 1) {for(int i = 1;i<=n;++i) s[i+n] = s[i]; n <<= 1;}  
		for(int i = 1;i <= n;i += 2) s2[(i + 1) >> 1] = f[s[i]][s[i+1]];  
		n >>= 1;  
		for(int i = 1;i<=n;++i) s[i] = s2[i];  
	}  
	putchar(s[1]);  
	return 0;  
}  
```  
  
#### D  
  
好 ad-hocs 的恶心题，Codeforces 的典型风格。  
  
考虑将某个角作为坐标基准点，框一个 $2 \times 2$ 的矩阵覆盖之。  
  
可以向角的那一边移动。  
  
向右上，三步一右上平移。  
  
难以分析，不写了艹。  
  
### Atcoder Regular Contest 108  
  
#### A  
  
根据数学知识，两个越接近，积越大。  
  
二分第一个数。然而需要注意可能溢出。  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
long long S,P;  
int main()  
{  
	scanf("%lld %lld",&S,&P);  
	long long up = S >> 1;  
	long long l = 1,r = up,mid,ans = -1;  
	while(l <= r)  
	{  
		mid = (l + r) >> 1;  
		if(mid > P / (S - mid) + 10) {r = mid - 1;continue;}  
		if(mid * (S - mid) <= P) l = mid + 1,ans = mid;  
		else r = mid - 1;  
	}  
	if(ans == -1 || ans * (S - ans) != P) puts("No");  
	else puts("Yes");  
	return 0;  
}  
```  
  
#### B  
  
不是很懂字符串，考虑胡乱暴力。每次删除后只往前跳两个点，继续暴力删。  
  
这样复杂度大概是对的，问题在于如何快速跳点。  
  
考虑并查集路径压缩？  
  
定义 $R(i)$ 为 $i$ 点往右第一个未被删除的点，$L(i)$ 同理，初始有 $L(i) = R(i) = i$.  
  
考虑删除点 $i$，会有：$L(i) = i-1$，$R(i) = i + 1$，这样应该就可以了吧？  
  
```cpp  
#include <cstdio>  
#include <algorithm>  
using namespace std;  
const int maxn = 2e5 + 100;  
int n,L[maxn],R[maxn];  
int findL(int x){return L[x] == x ? x : L[x] = findL(L[x]);}  
int findR(int x){return R[x] == x ? x : R[x] = findR(R[x]);}  
char s[maxn];  
inline void del(int x){L[x] = x - 1,R[x] = x + 1;}  
int main()  
{  
	scanf("%d",&n);  
	scanf("%s",s + 1);  
	for(int i = 0;i<=n + 10;++i) L[i] = R[i] = i;  
	int p1 = 1,p2 = 2,p3 = 3,res = 0;  
	while(p3 <= n)  
	{  
		if(s[p1] == 'f' && s[p2] == 'o' && s[p3] == 'x')  
		{  
			del(p3),del(p2),del(p1),res += 3;  
			for(int i = 1;i<=2;++i) if(p1) p1 = findL(p1 - 1);  
			p2 = findR(p1 + 1),p3 = findR(p2 + 1);  
			continue;  
		}  
		if(p3 >= n) break;  
		p1 = findR(p1 + 1),p2 = findR(p2 + 1),p3 = findR(p3 + 1);  
	}  
	printf("%d\n",n - res);  
	return 0;  
}  
```  
  
#### C  
  
考虑联通图是怎么样的，发现最小的联通图是一棵树。  
  
树上每个点总是有一个父亲，标记它的父亲就可以了。  
  
大力搜索，随便整一棵搜索树出来。原图联通，就不存在无解情况。  
  
```cpp  
#include <cstdio>  
const int maxn = 1e6 + 100;  
struct node  
{  
	int to,next,w;  
}E[maxn];  
int n,m,head[maxn],tot,c[maxn];  
inline void add(int x,int y,int w) { E[++tot].next = head[x],E[tot].to = y,E[tot].lab = w,head[x] = tot; }  
void dfs(int u)  
{  
	for(int p = head[u],v;p;p=E[p].next)  
	{  
		if(c[v = E[p].to]) continue;  
		c[v] = (E[p].w == c[u] ? c[u] % n + 1 : E[p].w),dfs(v);  
	}  
}  
int main()  
{  
	scanf("%d %d",&n,&m);  
	for(int i = 1,a,b,c;i<=m;++i) scanf("%d %d %d",&a,&b,&c),add(a,b,c),add(b,a,c);  
	col[1] = 1,dfs(1);  
	for(int i = 1;i<=n;++i) printf("%d\n",col[i]);  
	return 0;  
}  
```