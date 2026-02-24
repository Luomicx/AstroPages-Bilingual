---
title: 三维BFS
pubDatetime: 2025-03-26T22:17:54Z
description: 使用BFS的方法寻找最短路和出口
featured: false
draft: false
tags:
  - 算法
---













# 三维BFS



![image-20250327213836466](https://s2.loli.net/2025/03/27/nE2VbUKwyBgLx7S.png)

一个三维数组的BFS搜索，参考下边代码即可

```c++
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <iostream>
#include <queue>

using namespace std;
const int N = 110;
         //上 下 右  左 后 前
int dx[] = {0, 0, 1, -1, 0, 0};
int dy[] = {0, 0, 0, 0, -1, 1};
int dz[] = {1, -1,0, 0, 0, 0};

struct Node
{
    int x, y, z;  
}st, ed;
int l, r, c;
char g[N][N][N];
int d[N][N][N];
bool vis[N][N][N];

bool check(int x, int y, int z)
{
    if(x <= 0 || x > r) return false;
    if(y <= 0 || y > c) return false;
    if(z <= 0 || z > l) return false;
    if(d[x][y][z] != -1) return false;
    if(g[z][x][y] == '#') return false;
    
    return true;
}

void bfs()
{
    queue<Node> q;
    q.push(st);
    d[st.x][st.y][st.z] = 0;
    
    while(q.size())
    {
        auto t = q.front(); q.pop();
        int x = t.x, y = t.y, z = t.z;
        for(int i = 0;i < 6;i ++)
        {
            int nx = x + dx[i];
            int ny = y + dy[i];
            int nz = z + dz[i];
            if(check(nx, ny, nz))
            {
                q.push({nx, ny, nz});
                d[nx][ny][nz] = d[x][y][z] + 1;
            }
        }
    }
}

int main()
{
    while(cin >> l >> r >> c && l != 0)
    {
        for(int i = 1;i <= l;i ++)
            for(int j = 1;j <= r;j ++)
                for(int k = 1;k <= c; k ++)
                {
                    cin >> g[i][j][k]; 
                    if(g[i][j][k] == 'S')
                    {
                        st.z = i,st.x = j, st.y = k;
                    }
                    if(g[i][j][k] == 'E')
                    {
                        ed.z = i, ed.x = j, ed.y = k;
                    }
                }
        memset(d, -1, sizeof d);
        bfs();
        int ans = d[ed.x][ed.y][ed.z];
        if(ans == -1) puts("Trapped!");
        else printf("Escaped in %d minute(s).\n", ans);
        
        
    }
    
    return 0;
}
```

![image-20250327215453545](https://s2.loli.net/2025/03/27/w1Fd9nbPoerqhTx.png)

可以根据二叉树的性质来进行解决这个问题

其中我们的每层的二叉树的起点都是 $2^i$次方，i从0开始，所以对于每层遍历d的时候我们下标范围为$[i, 2^{d-1}]$  d表示层数，然后可以使用双指针算法，使得复杂度降到$O(n)$

```c++
#include <cstdio>
#include <cstring>
#include <iostream>
#include <algorithm>

using namespace std;

const int N = 1e5 + 10;

using LL = long long;

int n;
int a[N];

int main()
{
    scanf("%d", &n);
    for(int i = 1;i <= n;i ++) scanf("%d", &a[i]);
    
    LL maxs = -1e18; 
    int depth = 0; 
    
    for(int d = 1, i = 1;i <= n;i *= 2,d ++)
    {
        LL s = 0;
        for(int j = i;j < i + (1 << d - 1) && j <= n; j ++) s += a[j];
        if(s > maxs)
        {
            maxs = s;
            depth = d;
        }
    }
    
    printf("%d\n", depth);
    
    return 0;
}
```

