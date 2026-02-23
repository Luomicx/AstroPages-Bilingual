---
title: ACwing2025蓝桥杯每日一题
tags: []
copyright: true
date: 2025-03-28 11:17:54
permalink:
categories: 算法
description: 模拟,枚举
ogImage: http://img.wiretender.top/img/20250330161516653.webp
mathjax: true
---









# 第一题：模拟

![image-20250328105038163](https://s2.loli.net/2025/03/28/n6goGbz7dsOjxX5.png)

代码：

```c++
#include <iostream>

using namespace std;

const int N = 1010;

int n, Q;
int a[N][N], b[N][N], c[N][N];

int main()
{
    scanf("%d%d", &n, &Q);
    int res = 0;
    while (Q --)
    {
        int x, y, z;
        scanf("%d%d%d", &x, &y, &z);
        a[x][y] ++;
        b[y][z] ++;
        c[x][z] ++;
        if (a[x][y] >= n) res ++;
        if (b[y][z] >= n) res ++;
        if (c[x][z] >= n) res ++;

        printf("%d\n", res);
    }  

    return 0;
}
```

# 枚举

![image-20250328111231894](https://s2.loli.net/2025/03/28/eDLpvIjzhT3G1M9.png)

- 题意是在字符串 S 中寻找一个模式串出现的次数，并在剩下的串中允许修改一个字符，允许使这个字符串的次数加一
- 我们可以通过枚举的方法来求出相应的答案
- 时间复杂度为 $O(26\times 26 \times N)$

==代码如下==：

```c++
#include <iostream>
#include <cstring>
#include <algorithm>
#include <cstdio>
#include <vector>

using namespace std;

int n, f;
string s;

int main()
{
    cin.tie(0) -> sync_with_stdio(0);
    cin >> n >> f;
    cin >> s;
    vector<string> ans;
    for(char c1 = 'a'; c1 <= 'z'; c1 ++)
    {
        for(char c2 = 'a'; c2 <= 'z'; c2 ++)
        {
            if(c1 == c2) continue;
            int flag = 0;
            int cnt = 0;
            
            auto t = s;
            for(int i = 0;i + 2 < n;i ++)
            {
                if(t[i] == c1 && t[i + 1] == t[i + 2] && t[i + 1] == c2) 
                {
                    cnt ++;
                    t[i] = t[i + 1] = t[i + 2] = '#';
                }
                    
            }
            
            // 找剩余中的有可能的子串
            for(int i = 0;i + 2 < n;i ++)
            {
                if(t[i] == '#' || t[i + 1] == '#' || t[i + 2] == '#') continue;
                if((t[i] == c1 && t[i + 1] == c2) || 
                   (t[i] == c1 && t[i + 2] == c2) || 
                   (t[i + 1] == t[i + 2] && t[i + 1] == c2)) flag = 1;
            }
            
            if(cnt + flag >= f)
            {
                string t; t += c1, t += c2, t += c2;
                ans.push_back(t);
            }
        }
    }
     // 输出答案的个数及其各个字符串
    cout << ans.size() << endl;
    for(auto st : ans) cout << st << endl;
    return 0;
}
```

