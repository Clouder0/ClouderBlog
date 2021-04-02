---
title: "[笔记]高斯消元法与矩阵求逆"
date: 2020-09-10T04:45:00+08:00
slug: "gauss"
type: posts
tags:
  - gauss-elimination
  - math
  - matrix
categories:
  - Algorithm
---

## 高斯消元

>高斯消元法（Gauss-Jordan elimination）是求解线性方程组的经典算法，它在当代数学中有着重要的地位和价值，是线性代数课程教学的重要组成部分。

>高斯消元法除了用于线性方程组求解外，还可以用于行列式计算、求矩阵的逆，以及其他计算机和工程方面。

>夏建明等人之前提出了应用图形处理器 (GPU) 加速求解线性方程组的高斯消元法，所提出的算法与基于 CPU 的算法相比较取得更快的运算速度。二是提出各种变异高斯消元法以满足特定工作的需要。

高斯消元可以用于求解多元线性方程组，形如：

$$\begin{cases} a_{1,1}x_1 + a_{1,2}x_2 + \ldots + a_{1,n}x_n = b_1 \\ a_{2,1}x_1 + a_{2,2}x_2 + \ldots + a_{2,n}x_n = b_2 \\ \cdots \\ a_{n,1}x_1 + a_{n,2}x_2 + \ldots + a_{n,n}x_n = b_n \end{cases}$$



### 实现

将各项系数转化为矩阵。

$$\left( \begin{matrix} a_{1,1} & a_{1,2} & \cdots & a_{1,n} & b_1 \\ a_{2,1} & a_{2,2} & \cdots & a_{2,n} & b_2 \\ \cdots & \cdots & \cdots & \cdots & \cdots \\ a_{n,1} & a_{n,2} & \cdots & a_{n,n} & b_n \end{matrix} \right)$$

每列都进行消元后，每列中只有一行该列有值，其余行该列全是零。

什么叫消元？例如：

$$a_{1,1}x_1 + a_{1,2}x_2 + \ldots + a_{1,n}x_n = b_1$$

现在用它来消去其余行的 $x_1$ 系数，构造：

$$x_1 + \dfrac{a_{1,2}}{a_{1,1}}x_2 + \ldots + \dfrac{a_{1,n}}{a_{1,1}}x_n = \dfrac{b_1}{a_{1,1}}$$

然后对于任意的：

$$a_{i,1}x_1 + a_{i,2}x_2 + \ldots + a_{i,n}x_n = b_i$$

都可以做对应的减法，得到：

$(a_{i,1} - a_{i,1} \times 1)x_1 + (a_{i,2} - a_{i,1} \times \dfrac{a_{1,2}}{a_{1,1}})x_2 + \ldots = b_i - a_{i,1} \times \dfrac{b_1}{a_{1,1}}$

这样消去后，该行 $x_1$ 系数为 $0$.

使用每行的方程都消去一元，则每行最终系数都为：

$0x_1 + 0x_2 + \ldots + kx_i + \ldots + 0x_n = b'$

即可解出对应的 $x_i$.

---

在具体实现时，无须将用于消去的行的对应系数设为 $1$，只需要计算出消去相应所需的倍数即可。

$a_{1,1}x_1 + \ldots$

$a_{2,1}x_1 + \ldots$

只需要令 $t = \dfrac{a_{2,1}}{a_{1,1}}$，而减去一式的 $t$ 倍即可。

- 此外，若某列的系数全为 $0$，即该列无限制，说明方程组无唯一解。

- 消去 $x_i$ 时，可以选择 $a_{j,i}$ 最大的，即该列对应系数最大的行，作为用于消去的行，可以提高精度。

---

时间复杂度 $O(n^3)$.

```cpp
#include <algorithm>
#include <cmath>
#include <cstdio>
const int maxn = 110;
int n;
double a[maxn][maxn];
const double eps = 1e-8;
int main()
{
    scanf("%d", &n);
    for (int i = 1; i <= n; ++i) for (int j = 1; j <= n + 1; ++j) scanf("%lf", &a[i][j]);
    for (int i = 1; i <= n; ++i) // 选择列
    {
        //寻找系数最大行
        int maxx = i;
        for (int j = i + 1; j <= n; ++j) if (fabs(a[j][i]) > fabs(a[maxx][i])) maxx = j; 
        //将系数最大行交换至第 i 行
        for (int j = 1; j <= n + 1; ++j) std::swap(a[i][j], a[maxx][j]);
        //系数全零
        if (fabs(a[i][i]) < eps) goto fail;
        for (int j = 1; j <= n; ++j)
        {
            if (j == i) continue;
            //消元
            double t = a[j][i] / a[i][i];
            for (int k = i + 1; k <= n + 1; ++k) a[j][k] -= a[i][k] * t;
        }
    }
    //解一元一次方程
    for (int i = 1; i <= n; ++i) printf("%.2f\n", a[i][n + 1] / a[i][i]);
    return 0;
fail:
    puts("No Solution");
    return 0;
}
```

## 矩阵求逆

高斯消元法可以用于矩阵求逆。

什么叫矩阵的逆？对于矩阵 $A,B$，满足 $A \times B = B \times A = I_n$，其中 $I_n$ 为 $n$ 阶单位矩阵，则 $B$ 为 $A$ 的逆矩阵。

形象地说：

$$A = \begin{bmatrix} a_{1,1} & a_{1,2} & \cdots & a_{1,n} \\ \cdots & \cdots & \cdots & \cdots \\ a_{n,1} & a_{n,2} & \cdots & a_{n,n} \end{bmatrix}$$

$$B = \begin{bmatrix} b_{1,1} & b_{1,2} & \cdots & b_{1,n} \\ \cdots & \cdots & \cdots & \cdots \\ b_{n,1} & b_{n,2} & \cdots & b_{n,n} \end{bmatrix}$$

有：

$A \times B = \begin{bmatrix} 1 & 0 & \cdots & 0 \\ \cdots & \cdots & \cdots & \cdots \\ 0 & 0 & \cdots & 1 \end{bmatrix}$

则 $B$ 为 $A$ 的逆矩阵。

---

矩阵求逆的做法：

- 将 $A$ 与 $I$ 放在同一个矩阵中
- 对 $A$ 进行消元，将 $A$ 化为单位矩阵
- 此时原单位矩阵转化为 $A$ 的逆矩阵

可以发现，高斯消元后，原矩阵化为一个对角矩阵，即只有 $a_{i,i}$ 上有值。

$$\begin{bmatrix} a_{1,1} & 0 & \cdots & 0 \\ \cdots & \cdots & \cdots & \cdots \\ 0 & 0 & \cdots & a_{n,n} \end{bmatrix}$$

而每行除以 $a_{i,i}$ 后，即可化作单位矩阵。

而 $A$ 通过一系列的线性变化变成了 $I$，即单位矩阵，每次变化都可以写成矩阵的形式。

可以这样表达：$A \times P_1 \times P_2 \times \ldots P_m = I$

即 $A \times \prod \limits _{i=1}^m P_i = I$

根据逆矩阵定义，$A \times B = I$，所求即 $\prod \limits_{i=1}^m P_i$.

而将单位矩阵放在 $A$ 的右边，每次变化都会对单位矩阵同样变化。

$A \times B = I$

$I \times B = B$

即可求出 $A$ 的逆矩阵 $B$.

```cpp
#include <algorithm>
#include <cstdio>
const int maxn = 810;
const int p = 1e9 + 7;
#define ll long long
ll a[maxn][maxn];
int n;
void exgcd(ll a,ll b,ll &x,ll &y)
{
    if (b == 0) return (void)(x = 1, y = 0);
    exgcd(b, a % b, y, x), y -= (a / b) * x;
}
ll inv(ll x)
{
    ll k1,k2;
    exgcd(x,p,k1,k2);
    return ((k1 % p) + p) % p;
}
signed main()
{
    scanf("%d", &n);
    for (int i = 1; i <= n; ++i) for (int j = 1; j <= n; ++j) scanf("%lld", &a[i][j]);
    for (int i = 1; i <= n; ++i) a[i][i + n] = 1;
    for (int i = 1; i <= n; ++i)
    {
        int maxx = i;
        for (int j = i + 1; j <= n; ++j) if (a[j][i] > a[maxx][i]) maxx = j;
        if (i != maxx) std::swap(a[i], a[maxx]);
        if (a[i][i] == 0) goto fail;
        ll tmp = inv(a[i][i]);//求逆元
        for (int j = 1; j <= n; ++j)
        {
            if (i == j) continue;
            ll frac = (a[j][i] * tmp) % p;  // a[j][i] / a[i][i]
            for (int k = i; k <= 2 * n; ++k)
                a[j][k] = (((a[j][k] - frac * a[i][k]) % p) + p) % p;//消元
        }
        for (int j = 1; j <= 2 * n; ++j) a[i][j] = (a[i][j] * tmp) % p;//处理为 1
    }
    for (int i = 1; i <= n; puts(""), ++i) for (int j = 1; j <= n; ++j) printf("%lld ", a[i][j + n]);
    return 0;
fail:
    puts("No Solution");
    return 0;
}
```