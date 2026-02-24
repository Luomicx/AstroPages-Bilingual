---
title: ACwing算法笔记（1）
pubDatetime: 2025-03-27T13:44:40Z
description: ACwing算法笔记（1）
featured: false
draft: false
tags:
  - 算法
---

# 枚举

题目：![image-20250316211547064](https://s2.loli.net/2025/03/27/IJb1746pdxSRi5D.png)

代码：

```c++
#include <iostream>
#include <cstdio>
#include <algorithm>
#include <cstring>

using namespace std;

const int N = 10010;
const int INF = 100000000;

int n, a[N];

int main()
{
    cin >> n;
    for(int i = 1;i <= n;i ++) cin >> a[i];
    int ans = 0;
    for(int i = 1;i <= n;i ++)
    {
        int min_v = INF, max_v = -INF;
        for(int j = i;j <= n;j ++)
        {
            min_v = min(min_v, a[j]);
            max_v = max(max_v, a[j]);
            if(max_v - min_v == j - i ) ans ++;
        }
    }
    cout << ans << endl;
    return 0;
}
```

****

日期问题

![image-20250325183821889](https://s2.loli.net/2025/03/27/JBUyCe51a6qHzuo.png)

```c++
#include <iostream>
#include <cstdio>
using namespace std;

int months[13] = {0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};

bool check_valid(int year, int month, int day)
{
	if(month == 0 || month > 12) return false;
	if(day == 0) return false;
	if(month != 2)
	{
		if(day > months[month]) return false;
	}
	else
	{
		int leap = year % 100 != 0 && year % 4 == 0 || year % 400 == 0;
		if(day > 28 + leap) return false;	
	}
	return true;
}


int main()
{
	int a, b, c;
	scanf("%d/%d/%d", &a, &b, &c);
	
	for (int date = 19600101; date <= 20591231; date ++) 
	{
		int year = date / 10000;
		int month = date % 10000 / 100;
		int day = date % 100;
		if(check_valid(year, month, day))
		{
			if(year % 100 == a && month == b && day == c ||
			   year % 100 == c && month == a && day == b ||
			   year % 100 == c && month == b && day == a
			)
			printf("%d-%02d-%02d\n", year, month, day);
		}
	}
	
	return 0;
}
```

![image-20250325184832205](https://s2.loli.net/2025/03/27/IckMDEF5pY3N28L.png)

```c++
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>
using namespace std;

int get_seconds(int h, int m, int s)
{
	return h * 3600 + m * 60 + s;
}

int get_time()
{
	string line;
	getline(cin, line);
	if(line.back() != ')') line += "(+0)";
	
	int h1, m1, s1, h2, m2, s2, d;
	sscanf(line.c_str(), "%d:%d:%d %d:%d:%d (+%d)", &h1, &m1, &s1, &h2, &m2, &s2, &d);
	
	return get_seconds(h2, m2, s2) - get_seconds(h1, m1, s1) + d * 24 * 3600;
}


int main()
{
	int n; scanf("%d", &n);
	string line;
	getline(cin, line);
	while(n --)
	{
		int time = (get_time() + get_time()) / 2;
		int hour = time / 3600, minute = time % 3600 / 60, second = time % 60;
		printf("%02d:%02d:%02d", hour, minute, second);
	}
	return 0;
}
```

模拟

![image-20250325205133037](https://s2.loli.net/2025/03/27/Py9lw58rW2Zxeia.png)

```c++
#include <cstdio>
#include <iostream>
#include <algorithm>
#include <cstring>

#define x first
#define y second

using namespace std;
typedef pair<int, int> PII;
const int N = 1e5 + 10;

PII order[N];
int score[N], last[N];
bool st[N];

int main()
{
    int n, m, T;
    scanf("%d%d%d", &n, &m, &T);
    for (int i = 0; i < m;i ++) scanf("%d%d", &order[i].x, &order[i].y);
    sort(order, order + m); 
    // pair默认按照第一个关键字进行排序
    // 然后按照第二个关键字进行排序
    // 这里的排序先根据时间顺序进行排序，然后按照id进行排序
    // 枚举订单的数量
    for(int i = 0;i < m;)
    {
        int j = i;
        while(j < m && order[j] == order[i]) j ++;
        int t = order[i].x, id = order[i].y, cnt = j - i;
        i = j;
        
        score[id] -= t - last[id] - 1;
        if(score[id] < 0) score[id] = 0;
        if(score[id] <= 3) st[id] = false; // 这里处理的是t时刻之前的订单
        
        score[id] += cnt * 2;			   // 这里是处理的是t时刻的时候的订单
        if(score[id] > 5) st[id] = true;
        
        last[id] = t;
    }
    for(int i = 1; i <= n; i ++)
    {
        if(last[i] < T)
        {
            score[i] -= T - last[i];
            if(score[i] <= 3) st[i] = false;
        }
    }
    int res = 0;
    for(int i = 1;i <= n;i ++) res += st[i];
    
    printf("%d\n", res);
    
    return 0;
}
```

