---
title: 算法-DFS
pubDatetime: 2025-05-03T22:25:40Z
description: DFS算法笔记
featured: false
draft: false
tags:
  - 算法
---



# 飞机降落（十四届真题）

[P9241 [蓝桥杯 2023 省 B\] 飞机降落 - 洛谷](https://www.luogu.com.cn/problem/P9241)

```c++
#include <cstring>
#include <algorithm>
#include <unordered_set>
#include <iostream>
#include <bitset>

#define io ios::sync_with_stdio(0), cin.tie(0)
#define ll long long
using namespace std;

const int N = 15;

int n;
int t[N], d[N], l[N];
bitset<N> vis;
/*
	初始状态 (time=0)
	├─ 选择飞机1 (新time=max(0,t1)+l1)
	│  ├─ 选择飞机2 (新time=max(prev,t2)+l2)
	│  │  └─ 选择飞机3 → ✓ 成功叶子
	│  └─ 选择飞机3 (新time=max(prev,t3)+l3)
	│     └─ 选择飞机2 → ✓ 成功叶子
	├─ 选择飞机2 (新time=max(0,t2)+l2)
	│  ├─ 选择飞机1
	│  │  └─ 选择飞机3 → ✓ 成功叶子
	│  └─ 选择飞机3
	│     └─ 选择飞机1 → ✓ 成功叶子
	└─ 选择飞机3 (新time=max(0,t3)+l3)
	   ├─ 选择飞机1
	   │  └─ 选择飞机2 → ✓ 成功叶子
	   └─ 选择飞机2
	      └─ 选择飞机1 → ✓ 成功叶子 
*/
// 全排列问题
bool dfs(int u, int times) // u 表示成功降落了几架飞机
{
	if(u == n + 1) // 说明到最后一架飞机
	{
		return true;
	}
	
	for(int i = 1;i <= n;i ++)
	{
		if(!vis[i])
		{
			if(t[i] + d[i] < times) continue;
			vis[i] = true;
			int ti = max(times, t[i]) + l[i];
			if(dfs(u + 1, ti)) return true;
			vis[i] = false;
		}
	}
	return false;
}


int main()
{
	int _; cin >> _;
	while(_ -- )
	{
		cin >> n;
		for(int i = 1;i <= n; i ++) cin >> t[i] >> d[i] >> l[i];
		if(dfs(1, 0)) cout << "YES" << endl;
		else cout << "NO" << endl;
		vis.reset();
	}

	return 0;
}
```

# 搜索染色法

[P1162 填涂颜色 - 洛谷](https://www.luogu.com.cn/problem/P1162)

```c++
#include <cstring>
#include <algorithm>
#include <iostream>
#include <cstdio>
#include <bitset>
#include <vector>
#include <queue>
#include <utility>

#define io ios::sync_with_stdio(0), cin.tie(0)
#define ll long long
#define x first
#define y second

using namespace std;

const int N = 40;
const int mod = 1e9 + 7;

int n;
int a[N][N];
bitset<N> vis[N];

void dfs(int x, int y)
{
	if(x < 1 || x > n || y > n || y < 1) return ;
	if(a[x][y] != 0 || vis[x][y]) return;
	vis[x][y] = true;
	dfs(x + 1, y);
	dfs(x, y + 1);
	dfs(x - 1, y);
	dfs(x, y - 1);
}

void bfs(int x, int y)
{
	queue<pair<int, int>> q;
	q.push({x, y});
	vis[x][y] = true;
	
	int dx[] = {0, 0, -1, 1};
	int dy[] = {1, -1, 0, 0};
	
	while(!q.empty())
	{
		auto t = q.front(); q.pop();
		for(int i = 0;i < 4;i ++)
		{
			int nx = t.x + dx[i];
			int ny = t.y + dy[i];
			if(vis[nx][ny] || a[nx][ny] != 0) continue;
			if(nx < 1 || nx > n || ny < 1 || ny > n) continue;
			vis[nx][ny] = true;
			q.push({nx, ny});
		}
	}
}

int main()
{
	io;
	cin >> n;
	for(int i = 1;i <= n;i ++)
	{
		for(int j = 1;j <= n;j ++)
		{
			cin >> a[i][j];
		}
	}
	for(int i = 1;i <= n;i ++)
	{
		if(a[i][1] == 0) bfs(i, 1);
		if(a[i][n] == 0) bfs(i, n);
		if(a[1][i] == 0) bfs(1, i);
		if(a[n][i] == 0) bfs(n, i);
	}
	for(int i = 1;i <= n;i ++)
	{
		for(int j = 1;j <= n;j ++)
		{
			if(a[i][j] == 0 && !vis[i][j]) a[i][j] = 2;
		}
	}
	for(int i = 1;i <= n;i ++)
	{
		for(int j = 1;j <= n;j ++)
		{
			cout << a[i][j] << " ";
		}
		cout << endl;
	}
	return 0;
}
```

- 不知道编译器抽什么疯了，我的 `using PII = pair<int, int>;` 一直报错，只能写在里边了
- 代码中提供了 **DFS** 和 **BFS** 两种办法，供读者参考。
- 主要使用了 **染色法**，我们对整个边界上的 **0** 进行遍历来将不在 1 **包围圈** 的所有的 0 进行标记，然后我们对整个图进行遍历，我们对所有没有进行标记（也就是没有染色的 0） 进行变换操作变成 2，然后输出即可

# 奇怪的电梯（BFS最短路）

[P1135 奇怪的电梯 - 洛谷](https://www.luogu.com.cn/problem/P1135)

- dijkstra解法

```c++
#include <bits/stdc++.h>

#define io ios::sync_with_stdio(0), cin.tie(0)
#define ll long long
#define x first
#define y second

using namespace std;

const int N = 210;
const int mod = 1e9 + 7;

int n, st, ed;
int a[N];
int dist[N];
bitset<N> vis;
struct edge
{
	int u, w; // u 是出点， w 表示权值
	bool operator < (const edge& t) const 
	{
		return w > t.w;
	}
};
vector<edge> g[N];

void dijkstra(int s)
{
	memset(dist, 0x3f, sizeof dist);
	vis.reset();
	dist[s] = 0;
	priority_queue<edge>pq;
	pq.push({s, dist[s]});
	
	while(pq.size())
	{
		int x = pq.top().u;
		pq.pop();
		if(vis[x]) continue;
		vis[x] = true;
		
		for(auto &ed : g[x])
		{
			int y = ed.u;
			int w = ed.w;
			if(!vis[y] && dist[y] > dist[x] + w)
			{
				dist[y] = dist[x] + w;
				pq.push({y, dist[y]});
			}
		}
	}
}



int main()
{
	io;
	
	cin >> n >> st >> ed;
	// 这里我们利用相关的的数据进行建图
	for(int i = 1;i <= n;i ++)
	{
		int x; cin >> x;
		if(i + x <= n) g[i].push_back({i + x, 1});
		if(i - x > 0)  g[i].push_back({i - x, 1});
	}
	dijkstra(st);
	cout << (dist[ed] == 0x3f3f3f3f ? -1 : dist[ed]) << endl;
	return 0;
}
```



- BFS解法

```c++
#include <cstring>
#include <algorithm>
#include <iostream>
#include <cstdio>
#include <bitset>
#include <vector>
#include <queue>

#define io ios::sync_with_stdio(0), cin.tie(0)
#define ll long long
#define x first
#define y second

using namespace std;

const int N = 210;
const int mod = 1e9 + 7;

int n, st, ed;
int a[N];
bitset<N> vis;

void bfs(int s)
{
	queue<pair<int, int>> q;
	q.push({s, 0});
	vis[s] = true;
	
	while(!q.empty())
	{
		auto t = q.front(); q.pop();
		int p = t.x, step = t.y;
		if(p == ed)
		{
			cout << step << endl;
			return ;
		}
		if (p - a[p] > 0 && !vis[p - a[p]])
        {
            vis[p - a[p]] = true;
            q.push({p - a[p], step + 1});
        }
        if (p + a[p] <= n && !vis[p + a[p]])
        {
            vis[p + a[p]] = true;
            q.push({p + a[p], step + 1});
        }
	}
	cout << -1 << endl;	
}

int main()
{
	io;
	cin >> n >> st >> ed;
	for(int i = 1;i <= n;i ++) cin >> a[i];
	bfs(st);
	return 0;
}
```

