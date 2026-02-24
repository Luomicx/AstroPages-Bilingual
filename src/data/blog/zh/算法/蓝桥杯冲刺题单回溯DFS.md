---
title: 算法-蓝桥杯真题（3）
pubDatetime: 2025-04-03T22:25:40Z
description: 蓝桥杯真题（3）算法笔记
featured: false
draft: false
tags:
  - 算法
---

# 回溯法解决排列型枚举

![image-20250403162130962](http://imgbed.sut.qzz.io/img/20250403162131442.webp)

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <bitset>

using namespace std;

const int N = 10;

int n;
int ans[N];
bitset<N> vis;

void dfs(int dep)
{
	if(dep > n)
	{
		for(int i = 1;i <= n;i ++) cout << ans[i] << " \n"[i == n];
		return ;
	}

	for(int i = 1;i <= n;i ++)
	{
		if(vis[i]) continue;
		vis[i] = true;
		ans[dep] = i;
		dfs(dep + 1);
		vis[i] = false;
	}
}

int main()
{
	cin >> n;
	dfs(1);
	return 0;
}
```

# 递归实现指数型枚举

## 这里使用一个二进制集合枚举的方法

![image-20250403162224713](http://imgbed.sut.qzz.io/img/20250403162224817.webp)

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <bitset>

using namespace std;

const int N = 10;

int n;
int ans[N];
bitset<N> vis;

void dfs(int dep)
{
	if(dep > n)
	{
		for(int i = 1;i <= n;i ++) cout << ans[i] << " \n"[i == n];
		return ;
	}

	for(int i = 1;i <= n;i ++)
	{
		if(vis[i]) continue;
		vis[i] = true;
		ans[dep] = i;
		dfs(dep + 1);
		vis[i] = false;
	}
}

int main()
{
	cin >> n;
	dfs(1);
	return 0;
}
```

## 回溯做法

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <bitset>

using namespace std;

const int N = 17;

int n;
int ans[N];
bitset<N> st;

void dfs(int dep)
{
	if(dep > n)
	{
		for(int i = 1;i <= n;i ++)
		{
			if(st[i]) cout << i << " ";
		}
		cout << endl;
		return ;
	}
	else
	{
        // 当前位置是0
		st[dep] = false;
		dfs(dep + 1);
        // st[dep] = false;  这里的回溯可以不用做，下述同理
		// 当前位置是1
		st[dep] = true;
		dfs(dep + 1);
	}

}

int main()
{
	cin >> n;
	dfs(1);
	return 0;
}
```

# 递归实现组合型枚举

![image-20250403165502727](http://imgbed.sut.qzz.io/img/20250403165502842.webp)

```c++
#include <iostream>
#include <cstring>
#include <algorithm>
#include <cstdio>

using namespace std;

const int  N = 29;
int n, m; //0表示还没放置数字，1 表示选择当前的数字，2 表示不选择当前的数字

int way[N];

void dfs(int dep, int start)
{
   if(dep == m + 1)
   {
       for(int i = 1; i <= m;i ++) printf("%d ", way[i]);
       puts("");
       return ;
   }

   for(int i = start;i <= n;i ++)
   {
       way[dep] = i;
       dfs(dep + 1, i + 1);
       way[dep] = 0;
   }
}

int main()
{
    scanf("%d %d", &n, &m);

    dfs(1, 1);
    return 0;
}
```

# 美味的蛋糕（回溯法）

![image-20250403165530708](http://imgbed.sut.qzz.io/img/20250403165530823.webp)

```c++
#include <iostream>
#include <cstring>
#include <algorithm>
#include <cstdio>

using namespace std;
using ll = long long;

const int  N = 28;

int n, k;
int taste[N];
int ans = 0;
void dfs(int pos, int sum)
{
	// 表示选择到第 pos 个蛋糕，当前的总和为 sm
	// 对于每个美味值小于 k 的蛋糕我们有两种选择，选还是不选

	if(pos > n) {  // 已考虑所有蛋糕
        if(sum < k)
        	ans = max(ans, sum);
        return;
    }

    // 不选当前蛋糕
    dfs(pos + 1, sum);

    // 选当前蛋糕（前提是当前的美味值总和不超过k）
    if(sum + taste[pos] <= k) {
        dfs(pos + 1, sum + taste[pos]);
    }
}

int main()
{
    scanf("%d %d", &n, &k);
	for(int i = 1;i <= n;i ++) scanf("%d", &taste[i]);
    dfs(1, 0);
    cout << ans << endl;
    return 0;
}
```

# N皇后问题

![image-20250403165629550](http://imgbed.sut.qzz.io/img/20250403165629660.webp)

```c++
 #include <iostream>
#include <cstring>
#include <algorithm>
#include <cstdio>

using namespace std;
using ll = long long;

const int  N = 40;

int n;
int col[N], dg[N], udg[N];
int ans = 0;

void dfs(int u) // u 是当前行
{
	if(u == n)
	{
		ans ++;
		return ;
	}

	for(int i = 0;i < n;i ++) // i 枚举每一列
	{
    // 因为由一元线性函数得，我们可以用截距来表示每个对角线
    // y = x + b  ||  y = -x + b --> dg[x + y] && udg[y - x + n]（这里加个n是为了防止负数）
		if(!col[i] && !dg[u + i] && !udg[n - u + i])
		{
			col[i] = dg[u + i] = udg[n - u + i] = true;
			dfs(u + 1);
			col[i] = dg[u + i] = udg[n - u + i] = false;
		}
	}
}

int main()
{
    cin >> n;
	dfs(0);
	cout << ans;
    return 0;
}
```

# 五子棋博弈（二进制枚举）

![image-20250403212718696](http://imgbed.sut.qzz.io/img/20250403212718830.webp)

```c++
#include <iostream>
#include <cstdio>

using namespace std;


int n = 5;
int mp[5][5];
int a[25];


bool check()
{
	int pos = 0;
	for(int i = 0;i < 5;i ++)
		for(int j = 0;j < 5;j ++)
		{
			mp[i][j] = a[pos ++];
		}

	// 检查行和列
	for(int i = 0;i < 5;i ++)
	{
		int rowsum = 0, colsum = 0;
		for(int j = 0;j < 5;j ++)
		{
			rowsum += mp[i][j];
			colsum += mp[j][i];
		}
		if(rowsum == 0 || rowsum == 5 || colsum == 0 || colsum == 5) return false;
	}

	// 检查对角线
	int diag1 = 0, diag2 = 0;
	for(int i = 0;i < 5;i ++)
	{
		diag1 += mp[i][i];
		diag2 += mp[i][4 - i];
	}
	if(diag1 == 0 || diag1 == 5 || diag2 == 0 || diag2 == 5) return false;

	return true;
}

int main()
{
	cout << 3126376 << endl;
  return 0;
}
```

# 宝塔游戏（思维搜索）

![image-20250403212809197](http://imgbed.sut.qzz.io/img/20250403212809347.webp)

```c++
#include <iostream>
#include <cstring>
#include <cstdio>

using namespace std;
const int N = 10;

int dx1[] = {0, 2,2,1,3,3};
int dx2[] = {0, 1,4,2,2,3};
int dy1[] = {0, 2,2,3,2,1};
int dy2[] = {0, 3,3,1,2,4};

bool stx[N][N];
bool sty[N][N];
int g[N][N];
int n = 5;

bool check()
{
	// 检查dx两行
	for(int i = 1;i <= n;i ++)
	{
		int max1 = 0, ans = 0;
		for(int j = 1;j <= n;j ++)
		{
			if(g[i][j] > max1)
			{
				max1 = g[i][j];
				ans ++;
			}
		}
		if(ans != dy1[i]) return false;
		max1 = 0, ans = 0;
		for(int j = n;j >= 1;j --)
		{
			if(g[i][j] > max1)
			{
				max1 = g[i][j];
				ans ++;
			}
		}
		if(ans != dy2[i]) return false;
	}
	// 检查dx两列
	for(int j = 1;j <= n;j ++)
	{
		int max1 = 0, ans = 0;
		for(int i = 1;i <= n;i ++)
		{
			if(g[i][j] > max1)
			{
				max1 = g[i][j];
				ans ++;
			}
		}
		if(ans != dx1[j]) return false;
		max1 = 0, ans = 0;
		for(int i = n;i >= 1;i --)
		{
			if(g[i][j] > max1)
			{
				max1 = g[i][j];
				ans ++;
			}
		}
		if(ans != dx2[j]) return false;
	}

	return true;
}

void dfs(int x, int y)
{
	for(int i = 1;i <= n;i ++)
	{
		if(stx[y][i]) continue;
		if(sty[x][i]) continue;
		stx[y][i] = true;
		sty[x][i] = true;
		g[x][y] = i;
		if (y == n && x < n) {
            dfs(x + 1, 1);
        }
        else if (y < n && x <= n)dfs(x, y + 1);
        else if (x == n)
        {
            if (check())
            {
                for (int i = 1; i <= n; i++)
                    for (int j = 1; j <= n; j++)
                        printf("%d", g[i][j]);
            }
		}
		g[x][y] = 0;
		stx[y][i] = false;
		sty[x][i] = false;
	}
}

int main()
{
	dfs(1, 1);
	return 0;
}
```

# 数字接龙（2024省赛真题）

![image-20250403215313904](http://imgbed.sut.qzz.io/img/20250403215314036.webp)

```c++
#include <iostream>
#include <bitset>
using namespace std;

const int N = 12;

int n, k;
int g[N][N];
bitset<N> vis[N];
bool edge[N][N][N][N];
int dx[8] = {-1, -1, 0, 1, 1, 1, 0, -1}; // 定义8个方向的x坐标偏移
int dy[8] = {0, 1, 1, 1, 0, -1, -1, -1}; // 定义8个方向的y坐标偏移
string path;


bool dfs(int x, int y)
{
    // 边界条件
    if(x == n && y == n)
    {
    	return path.size() == n * n - 1;
    }

    vis[x][y] = true;
    for(int i = 0;i < 8;i ++)
    {
    	int nx = x + dx[i], ny = y + dy[i];
        // 1.检查是否越界
        // 2.检查是否走过
        // 3.检查是否满足回文的要求
        // 4.检查是否斜线有交叉
    	if(x < 1 || y < 1 || x > n || y > n) continue;
    	if(vis[nx][ny])continue;
    	if(g[nx][ny] != (g[x][y] + 1) % k) continue;
    	if(i % 2 && (edge[x][ny][nx][y] || edge[nx][y][x][ny])) continue;

    	edge[x][y][nx][ny] = true;
    	path += i + '0';

        // dfs到下一个点
    	if(dfs(nx, ny)) return true;

        // 回溯
    	path.pop_back();
    	edge[x][y][nx][ny] = false;
    }
    vis[x][y] = false;
    return false;
}

int main()
{
  cin >> n >> k;
  for(int i = 1;i <= n;i ++)
    for(int j = 1;j <= n;j ++)
      cin >> g[i][j];
  if(dfs(1, 1)) cout << path << endl;
  else cout << -1 << endl;
  return 0;
}

```
