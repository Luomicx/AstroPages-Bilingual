---
title: ACwing动态规划
tags: []
copyright: true
date: 2025-03-29 19:17:54
permalink:
categories: 算法
description: 动态规划算法
ogImage: http://img.wiretender.top/img/20250330154619864.webp
mathjax: true
---





# 动态规划（简单DP）

![image-20250316123553545](http://img.wiretender.top/img/20250330154619864.webp)

![image-20250316130222729](http://img.wiretender.top/img/20250330154723967.webp)

```c++
#include <iostream>
#include <algorithm>
#include <cstdio>
#include <cstring>

using namespace std;
const int N = 1005;

int f[N][N], w[N][N];

void solve()
{
    memset(f, 0, sizeof f);
    int r, c; cin >> r >> c;
    for(int i = 1;i <= r;i ++)
        for(int j = 1;j <= c;j ++)
            cin >> w[i][j];
    
    f[1][1] = w[1][1];
    for(int i = 1;i <= r;i ++)
        for(int j = 1;j <= c;j ++)
            f[i][j] = max(f[i - 1][j] + w[i][j], f[i][j - 1] + w[i][j]);
    
    cout << f[r][c] << endl;
}

int main()
{
    int _; cin >> _;
    while(_ --) solve();
    return 0;
}
```

![image-20250316141229576](http://img.wiretender.top/img/20250330154745553.webp)

```c++
#include <cstring>
#include <iostream>
#include <cstdio>
#include <algorithm>

using namespace std;

const int MOD = 1e9 + 7;

const int N = 52;

int n, m, k;
int w[N][N], f[N][N][13][14];

int main()
{
    cin >> n >> m >> k;
    for(int i = 1;i <= n;i ++)
        for(int j = 1;j <= m;j ++)
        {
            cin >> w[i][j];
            w[i][j] ++;
        }
        
    // Init
    f[1][1][1][w[1][1]] = 1;
    f[1][1][0][0] = 1;
    
    for(int i = 1;i <= n;i ++)
        for(int j = 1;j <= m;j ++)
        {
            if(i == 1 && j == 1) continue;
            for(int u = 0; u <= k;u ++)
                for(int v = 0; v <= 13;v ++)
                {
                    // 枚举我们不选择当前物品的集合的总数， 我们加上这一部分
                    int &val = f[i][j][u][v];
                    val = (val + f[i - 1][j][u][v]) % MOD;
                    val = (val + f[i][j - 1][u][v]) % MOD;
                    // 枚举下边取当前物品的集合的总数
                    // 条件一：需要当前的 u 也就是有物品,然后选择的物品的价值一定是 v，因为这								样才能被划为当前的集合
                    if(u > 0 && v == w[i][j])
                    {
                        // 条件二：这里要枚举所有小于v的物品的方案的数量，我们加上这一部分
                        for(int c = 0; c < v;c ++)
                        {
                            val = (val + f[i - 1][j][u - 1][c]) % MOD;
                            val = (val + f[i][j - 1][u - 1][c]) % MOD;
                        }
                    }
                }
        }
        
    int res = 0;
    for(int i = 0;i <= 13;i ++) res = (res + f[n][m][k][i]) % MOD;
    cout << res << endl;
    
    return 0;
}

```

# 多重背包二进制优化

```c++
#include <cstdio>
#include <algorithm>
#include <iostream>
#include <cstring>
using namespace std;
using ll = long long;
const int N = 110;

int n, m;
int v[N], w[N * 10], c[N];
int f[N];

int main()
{
    cin >> n >> m;
    /*for(int i = 1;i <= n;i ++) cin >> v[i] >> w[i] >> c[i];
	for(int i = 1;i <= n;i ++)
		for(int j = 0;j <= m;j ++)
			for(int k = 0;k <= c[i] && k * v[i] <= j;k ++)
			{
				f[i][j] = max(f[i][j], f[i - 1][j - k * v[i]] + k * w[i]);
			}	*/


    // 多重背包优化1
    // f[i][j]    = max(f[i-1][j], f[i-1][j-1*v[i]]+1*w[i], f[i-1][j-2*v[i]]+2*w[i], ...)
    // f[i][j-v]  = max(           f[i-1][j-1*v[i]],        f[i-1][j-2*v[i]]+1*w[i], ...)
    // 上边这个无法像无穷背包一样进行优化，所以我们进行一种二进制优化，类似打包的情况，这种情况下我们要将其所用的数组空间开大一些以确保可以存储完全

    int cnt = 0;
    for(int i = 1;i <= n;i ++)
    {
        int a, b, s;
        cin >> a >> b >> s;
        int k = 1;
        while(k <= s)
        {
            cnt ++;

            v[cnt] = a * k;
            w[cnt] = b * k;
            s -= k;
            k *= 2;
        }
        if(s > 0)
        {
            cnt ++;
            v[cnt] = a * s;
            w[cnt] = b * s;
        }
    }

    n = cnt;

    for(int i = 1;i <= n;i ++)
        for(int j = m;j >= v[i];j --)
            f[j] = max(f[j], f[j - v[i]] + w[i]);

    cout << f[m]; 


    //cout << f[n][m];


    return 0;
}
```

# 无穷背包解法

```c++
#include <cstdio>
#include <cstring>
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 1010;

int n, m;
int v[N], w[N];
int f[N][N];

int main()
{
    cin >> n >> m;
    for(int i = 1;i <= n;i ++) cin >> v[i] >> w[i];
    for(int i = 1;i <= m;i ++) f[0][i] = 0;

    for(int i = 1;i <= n;i ++)
        for(int j = 0;j <= m;j ++)
        {
            f[i][j] = f[i - 1][j];
            if(j >= v[i])
                // 这个公式是优化得到，所有无穷背包都可以这样写
                f[i][j] = max(f[i][j - v[i]] + w[i], f[i][j]); 
        }
    cout << f[n][m];
    return 0;
}
```

# 分组背包问题

![image-20250329201126167](http://img.wiretender.top/img/20250330154758535.webp)

- 思路：按照y总的模板来
- ![image-20250329202219940](http://img.wiretender.top/img/20250330154808377.webp)

```c++
#include <iostream>
#include <algorithm>
#include <cstdio>
#include <cstring>

using namespace std;

const int N = 110;

int n, m;
int v[N][N], w[N][N], s[N];
int f[N];

int main()
{
	cin >> n >> m;
	
	
	for(int i = 1;i <= n;i ++)
	{
		cin >> s[i];
		for(int j = 0;j < s[i]; j ++)
		{
			cin >> v[i][j] >> w[i][j];
		}
	}
	
	for(int i = 1;i <= n;i ++)
		for(int j = m;j >= 0;j --)
			for(int k = 0;k < s[i]; k ++)
			{
				if(v[i][k] <= j)
					f[j] = max(f[j], f[j - v[i][k]] + w[i][k]);
			}
	
	cout << f[m];
	return 0;
}
```

# 划分问题（无穷背包）

- 原题链接为 [0划分问题1 - 蓝桥云课](https://www.lanqiao.cn/problems/19917/learning/)
- ![image-20250329211145830](http://img.wiretender.top/img/20250330154817668.webp)

- 奉上解题思路：
- ![image-20250329211105418](http://img.wiretender.top/img/20250330154829301.webp)

```c++
// 二维dp
#include <iostream>
#include <algorithm>
#include <cstdio>
#include <cstring>

using namespace std;
const int N = 1010;
const int p = 1e9 + 7;

int f[N][N];

int main()
{
    int n; cin >> n;
    for(int i = 1;i <= n;i ++) f[i][0] = 1;
    for(int i = 1;i <= n;i ++)
        for(int j = 1;j <= n;j ++)
        {
            f[i][j] = f[i - 1][j];
            if(j >= i)
                f[i][j] = (f[i][j - i] + f[i][j]) % p;    
        }
    cout << f[n][n] << endl;
    return 0;
}
```

优化：

```c++
#include <iostream>
#include <algorithm>
#include <cstdio>
#include <cstring>

using namespace std;
const int N = 1010;
const int p = 1e9 + 7;

int f[N];

int main()
{
	int n; cin >> n;
	f[0] = 1;
	for(int i = 1;i <= n;i ++)
		for(int j = i;j <= n;j ++)
		{
				f[j] = (f[j - i] + f[j]) % p;	
	    }
	cout << f[n] << endl;
	return 0;
}
```

- 总结：和无穷背包的优化思路一致，可以看作**无穷背包**来进行思考

# 划分问题2（01背包）

![image-20250329211544616](http://img.wiretender.top/img/20250330154839517.webp)

**根据01背包和无穷背包的区别，直接改个*循环顺序*直接秒了**

**原因**：01背包是从上一层的状态转移过来，所以我们需要倒序遍历，这样可以保证我们之前的状态时被更新过的，而不是正序遍历造成重复遍历

```c++
#include <iostream>
#include <algorithm>
#include <cstdio>
#include <cstring>

using namespace std;
const int N = 1010;
const int p = 1e9 + 7;

int f[N];

int main()
{
	int n; cin >> n;
	f[0] = 1;
	for(int i = 1;i <= n;i ++)
		for(int j = n;j >= i;j --)
		{
				f[j] = (f[j - i] + f[j]) % p;	
	    }
	cout << f[n] << endl;
	return 0;
}
```

![image-20250329213752493](http://img.wiretender.top/img/20250330154853146.webp)

- 不开LL被卡了靠
- 基本思路看注释即可

```c++
#include <iostream>
using namespace std;
using ll = long long;
ll f[2023][11][2023]; // i, j, k 表示从前i个物品选择j个总体积和为k的方案数
// 有2022个物品，他们的体积即为编号本身，从2022个选取10个满足体积之和为2022的方案数
int main()
{
  // 边界条件：当体积为0，所有i都不选择，这也是一种选择，方案数为1
  for(int i = 0;i <= 2022;i ++) f[i][0][0] = 1;
  for(int i = 1;i <= 2022;i ++) //  枚举编号
    for(int j = 1;j <= 10;j ++) //  枚举个数
      for(int k = 1;k <= 2022;k ++) //  枚举体积
      {
        f[i][j][k] = f[i - 1][j][k]; // 不选择第i个物品
        if(k >= i)
          f[i][j][k] += f[i - 1][j - 1][k - i]; // 选择第i个物品
      }

  cout << f[2022][10][2022];
  return 0;
}
```

