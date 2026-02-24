---
title: 算法-BFS
pubDatetime: 2025-05-03T22:25:40Z
description: BFS算法笔记
featured: false
draft: false
tags:
  - 算法
---

# 图上 BFS（我目前只会 STL 哈哈哈）

- 简单的图上 BFS

![image-20250404213553144](http://imgbed.sut.qzz.io/img/20250404213553408.webp)

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <queue>

using namespace std;
using PII = pair<int, int>;

const int N = 410;

int dx[8]={-2,-2,2,2,1,-1,1,-1};
int dy[8]={-1,1,-1,1,2,-2,-2,2};

int n, m, x, y;
bool vis[N][N];
int dis[N][N];

bool inmp(int x, int y)
{
	return x >= 1 && x <= n && y >= 1 && y <= m;
}

void bfs(int sx, int sy)
{
	memset(dis, -1, sizeof dis);
	memset(vis, 0, sizeof vis);
	queue<PII> q;
	q.push({sx, sy});
	vis[sx][sy] = true;
	int step = 0;
	dis[sx][sy] = step;
	while(q.size())
	{
		int x = q.front().first;
		int y = q.front().second;
		q.pop();
		vis[x][y] = true;
		for(int i = 0;i < 8 ;i ++)
		{
			int nx = x + dx[i];
			int ny = y + dy[i];
			if(inmp(nx, ny) && !vis[nx][ny] && dis[nx][ny] == -1)
			{
				vis[nx][ny] = true;
				dis[nx][ny] = dis[x][y] + 1;
				q.push({nx, ny});
			}
		}
	}
}

int main()
{
	cin >> n >> m >> x >> y;
	bfs(x, y);
	for(int i = 1;i <= n;i ++)
	{
		for(int j = 1;j <= m;j ++)
			printf("%-5d", dis[i][j]);
		puts("");
	}


	return 0;
}
```

- 难点吗？有点意思，不会下象棋的有苦头了，（不会写方位数组（bushi））

# 迷宫（经典搜索习题）

![image-20250404215345823](http://imgbed.sut.qzz.io/img/20250404215346089.webp)

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <queue>

using namespace std;
using PII = pair<int, int>;

const int N = 7;
// up down left right
int dx[4] = {-1, 1, 0, 0};  // 上、下、左、右
int dy[4] = {0, 0, -1, 1};

int n, m, t;
int sx, sy, fx, fy;
bool vis[N][N];
int g[N][N];

int ans = 0;

bool inmp(int x, int y)
{
	return x >= 1 && x <= n && y >= 1 && y <= m;
}

void dfs(int x, int y)
{
	// 到达终点
	if(x == fx && y == fy)
	{
		ans ++;
		return ;
	}
	vis[x][y] = true;
	for(int i = 0;i < 4;i ++)
	{

		int nx = x + dx[i];
		int ny = y + dy[i];

		if(!inmp(nx, ny)) continue;
		if(g[nx][ny] == 1) continue; // 有障碍就跳过
		if(vis[nx][ny]) continue;

		// 都合法之后就处理下一个要去的点
		dfs(nx, ny);
	}

    // 这里四个方向的dfs都处理完之后要恢复我们的（x，y）的状态
	vis[x][y] = false;
}


int main()
{
	cin >> n >> m >> t;
	cin >> sx >> sy >> fx >> fy;
	memset(g, 0, sizeof g);
	memset(vis, 0, sizeof vis);
	for(int i = 1;i <= t;i ++)
	{
		int x, y; cin >> x >> y;
		// g[x][y] 表示这里有障碍
		g[x][y] = 1;
	}

	dfs(sx, sy);
	if(sx == fx && sy == fy)
	{
		cout << 1;
		return 0;
	}
	cout << ans;
	return 0;
}
```

- **易错点**：在当前的这一层的 **dfs** 中只关心当前状态，即我们这层的 **x** 和 **y**。 刚开始写的时候一直在对 **nx** 和 **ny** 操作，这是 **错误** 的。

# BFS 解决最少步骤

![image-20250404221951634](http://imgbed.sut.qzz.io/img/20250404221951799.webp)

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <queue>
#include <unordered_map>

using namespace std;
using PII = pair<int, int>;

const int N = 1e5 + 10;

int n, k;
int ans;
int bfs(int n, int k)
{
	queue<int> q;
	unordered_map<int, int> dist;
	q.push(n);
	dist[n] = 0;

	while(q.size())
	{
		// 三种状态
		int t = q.front();
		q.pop();
		int op[3] = {t + 1, t - 1,t * 2};
		for(int i = 0;i < 3;i ++)
		{
			int next = op[i];

			if(next == k) return dist[t] + 1;

			if(next >= 0 && next < N && !dist.count(next))
			{
				dist[next] = dist[t] + 1;
				q.push(next);
			}
		}
	}
	return -1;
}

int main()
{
	cin >> n >> k;
	cout << bfs(n, k);

	return 0;
}
```

## 同类型题：宇宙答案（来源 Eriktse 网站）

![image-20250404222054423](http://imgbed.sut.qzz.io/img/20250404222054602.webp)

```c++
//Code Here.
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
const int N = 2e6 + 9;

int bfs(ll x, ll y)
{
	vector<int> f(N, 1); // f[i] 表示从x走到i的最小距离
	queue<int> q;
	q.push(x);
	f[x] = 0; // 初始化当前的第一个x最小距离为0，也就是不做任何操作
	while(q.size()) // 利用队列进行bfs
	{
		int u = q.front(); q.pop(); // 将下一个走的点加入队列，弹出队首表示处理过了
		for(int i = 1;i <= 3; i++)
		{
			int v;  // 枚举下一个操作的点

			if(i == 1) v = u * 1.6;
			else if(i == 2) v = u * 0.6;
			else if(i == 3) v = u + 1;

			if(v > 2e6 || f[v] != 1) continue;
			// u -> v
			f[v] = f[u] + 1; // 表示从u到v已经进行了一步操作
			if(v == y) return f[v];
			q.push(v);
		}
	}
	return -1;
}
void solve()
{
	ll x, y; cin >> x >> y;
	cout << bfs(x, y) << '\n';
}
int main()
{
	int _; cin >> _;
	while(_--) solve();
	return 0;
}
```

# BFS 解答迷宫最短路问题（国赛真题）

![image-20250405101525581](http://imgbed.sut.qzz.io/img/20250405101525766.webp)

- 这里参考题解使用了 **反向搜图** 的方法， 这个方法很妙，让人惊叹哈哈哈，通过从终点出发，通过 bfs 来更新我们从终点到每一个图上的点的最短路径的长度，特别巧妙地求出了答案，如果使用起点作为 **bfs 的起点**，我们得到的 dist 数组就是我们从起点出发到每一个点的最短距离，这与我们的题意相悖。

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <queue>
#include <vector>

using namespace std;
using PII = pair<int, int>;

const int N = 2010;

int dx[] = {0, 0, -1, 1};
int dy[] = {1, -1, 0, 0};

int n, m;
int dist[N][N];
bool vis[N][N], isdoor[N][N];
vector<PII> door[N][N];


void bfs()
{
	memset(dist, 0x3f, sizeof dist);
	dist[n][n] = 0;
	queue<PII> q;
	q.push({n, n});

	while(q.size())
	{
		auto t = q.front();
		q.pop();

		int x = t.first;
		int y = t.second;
		// 有传送门的转移方法
		if(isdoor[x][y])
		{
			for(auto next : door[x][y])
			{
				// get every point
				int ddx = next.first, ddy = next.second;
				if(dist[ddx][ddy] > dist[x][y] + 1)
				{
					dist[ddx][ddy] = dist[x][y] + 1;
					q.push({ddx, ddy});
				}
			}
		}
		// 普通的转移方法
		for(int i = 0;i < 4;i ++)
		{
			int nx = t.first + dx[i];
			int ny = t.second + dy[i];
			if(nx < 1 || nx > n || ny < 1 || ny > n) continue;
			// if(vis[x][y]) continue; // bfs保证每个点只会走一次，所以不用进行标记
			if(dist[nx][ny] > dist[x][y] + 1)
			{
				dist[nx][ny] = dist[x][y] + 1;
				q.push({nx, ny});
			}
		}
	}
}


int main()
{
	scanf("%d%d", &n, &m);

	for(int i = 1;i <= m;i ++)
	{
		int a, b, c, d;
		scanf("%d%d%d%d", &a, &b, &c, &d);
		door[a][b].push_back({c, d});
		door[c][d].push_back({a, b});
		isdoor[a][b] = isdoor[c][d] = true;
	}

	bfs();
	long long sum = 0;
	for(int i = 1;i <= n;i ++)
		for(int j = 1;j <= n;j ++)
		{
			sum += dist[i][j];
		}

	printf("%.2lf", double((1.0 * sum) / (n * n) ) );
	return 0;
}
```

# BFS思维题

![image-20250405111221403](http://imgbed.sut.qzz.io/img/20250405111221620.webp)

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <queue>
#include <vector>
#define x first
#define y second
#define endl '\n'

using namespace std;
using PII = pair<int, int>;

const int N = 310;

int dx[] = {0, 0, -1, 1};
int dy[] = {1, -1, 0, 0};

int n, m;
int dist[N][N];
int g[N][N];
bool path[N][N];

void bfs(int sx, int sy)
{
	memset(dist, 0x3f, sizeof dist);
	queue<PII> q;
	dist[sx][sy] = 1;
	q.push({sx, sy});
	while(q.size())
	{
		auto t = q.front();
		q.pop();
		for(int i = 0;i < 4;i ++)
		{
			int nx = t.x + dx[i], ny = t.y + dy[i];
			if(nx < 1 || nx > n || ny < 1 || ny > m) continue;
			if(g[nx][ny] == 0) continue;
			if(dist[nx][ny] > dist[t.x][t.y] + 1)
			{
				dist[nx][ny] = dist[t.x][t.y] + 1;
				q.push({nx, ny});
			}
		}
	}
}

int main()
{
	cin >> n >> m;
	int sx, sy, ex, ey;
	cin >> sx >> sy >> ex >> ey;
	// 贪心的思路，我们只要找出一条最短路即可，然后将其他所有的1变成0然后就可以获得最多的金币
	for(int i = 1;i <= n;i ++)
	{
		string path;
		cin >> path;
		for(int j = 1;j <= m;j ++)
		{
			g[i][j] = path[j - 1] - '0';
		}
	}

	// for(int i = 1;i <= n;i ++)
		// for(int j = 1;j <= m;j ++) cout << g[i][j] << " \n"[j == m];

	bfs(sx, sy);
	// cout << dist[ex][ey] << endl;
	int ans = dist[ex][ey];
	cout << n * m - ans << endl;
	return 0;
}
```

- **易错点**：一定要处理字符串的偏移问题，如果要使用`1-based`索引就要处理字符串偏移，或者直接使用`0-based`的索引。

# 岛屿个数（2023省赛真题）

![image-20250405111627459](http://imgbed.sut.qzz.io/img/20250405111627593.webp)

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <queue>
#include <vector>
#define x first
#define y second
#define endl '\n'

using namespace std;
using PII = pair<int, int>;

const int N = 60;

int dx[] = {0, 0, -1, 1};
int dy[] = {1, -1, 0, 0};

int ddx[] = {0, 0, -1, 1, -1, 1, -1, 1};
int ddy[] = {1, -1, 0, 0, 1, 1, -1, -1};

int g[N][N];
int ans;
int n, m;

bool vis_sea[N][N];
bool vis_road[N][N];

bool inmp(int x, int y)
{
	return  x >= 0 && y >= 0 && x < n && y < m;
}

void road_bfs(int sx, int sy)
{
	queue<PII> q;
	q.push({sx, sy});
	vis_road[sx][sy] = true;
	while(q.size())
	{
		auto t = q.front();
		q.pop();
		for(int i = 0;i < 4;i ++)
		{
			int nx = t.x + dx[i], ny = t.y + dy[i];
			if(inmp(nx, ny) && g[nx][ny] == 1 && !vis_road[nx][ny])
			{
				vis_road[nx][ny] = true;
				q.push({nx, ny});
			}
		}
	}
}

void sea_bfs(int sx, int sy)
{
	queue<PII> q;
	q.push({sx, sy});
	vis_sea[sx][sy] = true;
	while(q.size())
	{
		auto t = q.front();
		q.pop();
		for(int i = 0;i < 8;i ++)
		{
			int nx = t.x + ddx[i], ny = t.y + ddy[i];
			// 如果是海洋
			if(inmp(nx, ny) && g[nx][ny] == 0 && !vis_sea[nx][ny])
			{
				vis_sea[nx][ny] = true;
				q.push({nx, ny});
			}
			// 如果是陆地
			if(inmp(nx, ny) && g[nx][ny] == 1 && !vis_road[nx][ny])
			{
				ans ++;
				// 以当前点来扩展整个相连的岛屿
				road_bfs(nx, ny);
			}
		}

	}
}

void solve()
{
	cin >> n >> m;
	// init
	ans = 0;
	for(int i = 0;i < n;i ++)
		for(int j = 0;j < m;j ++)
			vis_sea[i][j] = vis_road[i][j] = false;

	// read map
	for(int i = 0; i < n;i ++)
	{
		string s; cin >> s;
		for(int j = 0;j < m;j ++)
		{
			g[i][j] = s[j] - '0';
		}
	}

	// check borader
	// 检查边界，如果边界一圈都是陆地，那么所有中间的岛屿都是子岛屿，这时候岛屿为1
	bool flag = false;
	for(int i = 0;i < n;i ++)
		for(int j = 0;j < m; j ++)
		{
			if(i == 0 || i == n - 1 || j == 0 || j == m - 1)
			{
				// 有一块海洋即可
				if(vis_sea[i][j] == false && g[i][j] == 0)
				{
					flag = true;
					sea_bfs(i, j);
				}
			}
		}
	if(!flag) ans = 1;
	cout << ans << endl;
}

int main()
{
	int t; cin >> t;
	while(t --) solve();
	return 0;
}
```

# 星际旅行

![image-20250405141906021](http://imgbed.sut.qzz.io/img/20250405141906323.webp)

- vector实现邻接表

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <queue>
#include <vector>
#include <bitset>
#define endl '\n'

using namespace std;
using PII = pair<int, int>;

const int N = 1010;

struct edge
{
    int x, w;
    bool operator < (const edge &u) const{
        return w > u.w;
    }
};

int d[N][N];
int n, m, q;
int ans;
vector<edge> g[N];
bool vis[N];

void dijkstra(int st)
{
    memset(vis, 0, sizeof vis);
    d[st][st] = 0;
    priority_queue<edge> pq;

    pq.push({st, d[st][st]});

    while(!pq.empty())
    {
        int x = pq.top().x; pq.pop();

        if(vis[x]) continue;
        vis[x] = true;

        for(auto &next : g[x])
        {
            int y = next.x;
            int w = next.w;
            if(!vis[y] && d[st][y] > d[st][x] + w)
            {
                d[st][y] = d[st][x] + w;
                pq.push({y, d[st][y]});
            }
        }
    }
}

int main()
{
    cin >> n >> m >> q;
    memset(d, 0x3f, sizeof d);
    for(int i = 1;i <= m;i ++)
    {
        int a, b;
        cin >> a >> b;
        g[a].push_back({b, 1});
        g[b].push_back({a, 1});
    }

    for(int i = 1;i <= n;i ++)
    {
        dijkstra(i);
    }

    for(int j = 1;j <= q;j ++)
    {
        int a, b; cin >> a >> b;
        for(int i = 1;i <= n;i ++)
        {
            if(d[a][i] <= b) ans ++;
        }
    }

    printf("%.2lf", (double)ans / q);
    return 0;
}
```

- 数组模拟邻接表实现

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <queue>
#include <vector>
#include <bitset>
#define endl '\n'

using namespace std;
using PII = pair<int, int>;

const int N = 1010;

struct edge
{
	int u, w;
	bool operator < (const edge &u) const{
		return w > u.w;
	}
};

int d[N][N];
int n, m, q;
int ans;
int h[N], e[N * 2], ne[N * 2], idx;
int w[N * 2];
bool vis[N];


void add(int a, int b, int c)
{
	e[idx] = b;
	ne[idx] = h[a];
	w[idx] = c;
	h[a] = idx ++;
}

void dijkstra(int st)
{
	memset(vis, 0, sizeof vis);
	d[st][st] = 0;
	priority_queue<edge> pq;
	pq.push({st, d[st][st]});
	while(pq.size())
	{
		int x = pq.top().u;
		pq.pop();

		if(vis[x]) continue;
		vis[x] = true;

		for(int i = h[x]; i != -1;i = ne[i])
		{
			int j = e[i];
			if(!vis[j] && d[st][j] > d[st][x] + w[i])
			{
				d[st][j] = d[st][x] + w[i];
				pq.push({j, d[st][j]});
			}
		}
	}
}

int main()
{
	memset(h, -1, sizeof h);
	memset(d, 0x3f, sizeof d);
	cin >> n >> m >> q;
	for(int i = 1;i <= m;i ++)
	{
		int a, b;
		cin >> a >> b;
		add(a, b, 1);
		add(b, a, 1);
	}

	for(int i = 1;i <= n;i ++)
	{
		dijkstra(i);
	}

	for(int j = 1;j <= q;j ++)
	{
		int a, b; cin >> a >> b;
		for(int i = 1;i <= n;i ++)
		{
			if(d[a][i] <= b) ans ++;
		}
	}

	printf("%.2lf", (double)ans / q);
	return 0;
}
```
