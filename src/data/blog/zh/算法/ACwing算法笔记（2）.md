---
title: ACwing算法笔记（2）双指针 BFS 图论
pubDatetime: 2025-03-27T13:45:40Z
description: ACwing算法笔记（2）双指针 BFS 图论
featured: false
draft: false
tags:
  - 算法
---







滑动窗口

![image-20250325210828110](https://s2.loli.net/2025/03/27/tRBh3PajwFvZpeH.png)

```c++
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <iostream>

#define x first
#define y second

using namespace std;
const int N = 1e5 + 10;
typedef pair<int, int> PII;

int n, d, k;
PII records[N];
int cnt[N];
bool st[N];

int main()
{
    scanf("%d%d%d", &n, &d, &k);
    for(int i = 0;i < n;i ++) scanf("%d%d", &records[i].x, &records[i].y);
    sort(records, records + n);
    
    for(int i = 0, j = 0;i < n;i ++) // i 是右指针 j 是左指针
    {
        int id = records[i].y;
        cnt[id] ++;
        
        while(records[i].x - records[j].x >= d)
        {
            cnt[records[j].y] --;
            j ++;
        }
        
        if(cnt[id] >= k) st[id] = true;
    }
    for(int i = 0;i < 1e5 + 10;i ++)
    {
        if(st[i]) printf("%d\n", i);   
    }
    return 0;
}
```



BFS习题

![image-20250325213749132](https://s2.loli.net/2025/03/27/eoLFnAWUylPMt4a.png)

```c++
#include <cstdio>
#include <cstring>
#include <iostream>
#include <algorithm>
#include <queue>

#define x first
#define y second

using namespace std;
using PII = pair<int, int>;
const int N = 210;

int n, m;
char g[N][N];
int dist[N][N];

int bfs(PII start, PII end)
{
    queue<PII> q;
    memset(dist, -1, sizeof dist);
    
    dist[start.x][start.y] = 0;
    q.push(start);
    
    int dx[4] = {0, -1, 1, 0}, dy[4] = {1, 0, 0, -1};
    while(q.size())
    {
        auto s = q.front(); q.pop();
        for(int i = 0;i < 4;i ++)
        {
            int x = s.x + dx[i], y = s.y + dy[i];
            if(x < 0 || x >= n || y < 0 || y >= m) continue;
            if(g[x][y] == '#') continue;
            if(dist[x][y] != -1) continue;
            
            dist[x][y] = dist[s.x][s.y] + 1;
            
            if(end == make_pair(x, y)) return dist[x][y];
            
            q.push({x, y});
        }
    }
    return -1;
}

int main()
{
    int t; scanf("%d", &t);
    while(t --)
    {
        scanf("%d%d", &n, &m);
        for(int i = 0;i < n;i ++) scanf("%s", g[i]);
        
        PII start, end;
        for(int i = 0;i < n;i ++)
            for(int j = 0;j < m;j ++)
            {
                if(g[i][j] == 'S') start = {i, j};
                else if(g[i][j] == 'E') end = {i, j};
            }
        
        int distance = bfs(start, end);
        if(distance == -1) puts("oop!");
        else printf("%d\n", distance);
    }
    return 0;
}
```

交换瓶子

![image-20250327195009142](https://s2.loli.net/2025/03/27/6bia3rkBZleQuUT.png)

- 思路很巧妙，我们构想一个环来解决这个问题，本质是一个图论的问题
- 对于每个数组下标，我们定义为顶点的值，而我们将其所对应的值定义为顶点的出点，
- 也就是说，我们的最终目的是要将所有的环破开成为n个单独的环，其中每个环都指向自己，那么我们首先应该进行初始化操作，根据题目给我们的数据，
- 所以我们只要进行一个判环的操作即可完成
- 具体请看下图

![image-20250327202227640](https://s2.loli.net/2025/03/27/7WnBjG413gt96oV.png)

- 根据情况的分析我们得知，我们每次交换同一个环内的点必然会增加一个环，所以我们每次操作都会增加一次环，可以证明我们的交换次数是大于等于 n - 环 的个数的，所以我们的答案就是 n - k。

代码如下：

```c++
#include <cstdio>
#include <iostream>
#include <algorithm>
#include <cstring>

const int N = 10010;
using namespace std;
int n, a[N];
bool st[N];

int main()
{
    cin >> n;
    for(int i = 1;i <= n;i ++) cin >> a[i];
    int cnt = 0;
    for(int i = 1;i <= n;i ++)
        if (!st[i])
        {
            cnt ++;
            for(int j = i; !st[j]; j = a[j])
                st[j] = true;
        }
    
    cout << n - cnt << endl;
    return 0;
} 
```

时间复杂度为$O(n)$

