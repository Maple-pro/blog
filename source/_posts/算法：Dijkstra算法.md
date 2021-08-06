---
title: 算法：Dijkstra算法
date: 2021-04-13 10:53:32
tags: [图, dfs, 最短路]
categories: [算法]
description: Dijkstra + DFS的板子
mathjax: true
---

当涉及到比较复杂的第二标尺第三标尺时，可以采用Dijkstra + DFS的模板。

即，现在Dijkstra算法中记录下所有最短路径（只考虑距离），然后从这些最短路径中选出一条第二标尺最优的路径（通过DFS）。

复杂度：$O(V^2 + E)$

若采用堆优化来降低复杂度，即使用STL中的 `priority_que`。此时时间复杂度为 $O(VlogV + E)$ 

# 1. 模板

1. 使用Dijkstra算法记录所有最短路径

注意：

- 找到更优的前驱时，清空`pre[v]`
- 找到相同距离的路径时，不用清空`pre[v]`

```c++
// d[maxv]存储每个节点到起点的最短路径
// G[maxv][maxv]存储整个图
vector<int> pre[maxv];
void Dijkstra(int s) // s为起点
{
    fill(d, d + maxv, INF);
    d[s] = 0;
    for (int i = 0; i < n; i++) {
        int u = -1, MIN = INF;
        for (int j = 0; j < n; j++) {
            if (!vis[j] && d[j] < MIN) {
                u = j;
                MIN = d[j];
            }
        }
        if (u == -1) return;
        vis[u] = true;
        for (int v = 0; v < n; v++) {
            if (!vis[v] && G[u][v] != INF) {
                if (G[u][v] + d[u] < d[v]) {
                    d[v] = d[u] + G[u][v];
                    pre[v].clear();
                    pre[v].push_back(u);
                }
                else if (G[u][v] + d[u] == d[v]) {
                    pre[v].push_back(u);
                }
            }
        }
    }
}
```

2. 遍历所有最短路径，找出一条使第二标尺最优的路径

```c++
// 作为全局变量的第二标尺最优质optValue
// 记录最优路径的数组path（使用vector）
// 临时记录DFS遍历到叶子节点时的路径tmpPath（使用vector）
// st为起始节点
int optValue;
vector<int> pre[maxv];
vector<int> path;
vector<int> tmpPath;
void DFS(int v) // v为当前访问节点
{
    if (v == st) {
        tmpPath.push_back(v);
        int value;
        // TODO: 计算路径中tmpPath上的value值
        if (value 优于 optValue) {
            optValue = value;
            path = tmpPath;
        }
        tmpPath.pop_back();
        return;
    }
    tmpPath.push_back(v);
    for (int i = 0; i < pre[v].size(); i++) {
        DFS(pre[v][i]);
    }
    tmpPath.pop_back(); // 遍历完所有前驱节点，将当前节点v删除
}
```

计算路径中`tmpPath`上的`value`值（以计算边权和和点权和为例）：

>  注意要倒序访问节点

```c++
// 边权和
int value = 0;
for (int i = tmpPath.size() - 1; i > 0; i--) {
    int id = tmpPath[i];
    int next = tmpPath[i - 1];
    value += V[id][idNext];
}

// 点权和
int value = 0;
for (int i = tmpPath.size() - 1; i >= 0; i--) {
    int id = tmpPath[i];
    value += W[id];
}
```

# 2. 例题

[PAT-A1030 Travel Plan](https://pintia.cn/problem-sets/994805342720868352/problems/994805464397627392)

```c++
#include <iostream>
#include <string.h>
#include <vector>
using namespace std;

const int INF = 0x3f3f3f3f;
const int maxn = 505;
int d[maxn][maxn];
int c[maxn][maxn];
vector<int> pre[maxn];
int dist[maxn];
int vis[maxn] = {0};
int N, M, S, D;
int optValue = INF;
vector<int> path;
vector<int> tmpPath;

void Dijkstra()
{
	memset(dist, 0x3f, sizeof(dist));
	dist[S] = 0;
	for (int i = 0; i < N; i++) {
		int now = -1;
		int MIN = INF;
		for (int j = 0; j < N; j++) {
			if (!vis[j] && dist[j] < MIN) {
				now = j;
				MIN = dist[j];
			}
		}
		if (now == -1) return;
		vis[now] = 1;
		for (int j = 0; j < N; j++) {
			if (!vis[j]) {
				if (dist[now] + d[now][j] < dist[j]) {
					dist[j] = dist[now] + d[now][j];
					pre[j].clear();
					pre[j].push_back(now);
				}
				else if (dist[now] + d[now][j] == dist[j]) {
					pre[j].push_back(now);
				}
			}
		}
	}
}

void DFS(int now)
{
	if (now == S) {
		tmpPath.push_back(now);
		int value = 0;
		for (int i = tmpPath.size() - 1; i > 0; i--) {
			int id = tmpPath[i];
			int next = tmpPath[i - 1];
			value += c[id][next];
		}
		if (value < optValue) {
			path = tmpPath;
			optValue = value;
		}
		tmpPath.pop_back();
		return;
	}
	tmpPath.push_back(now);
	for (int i = 0; i < pre[now].size(); i++) {
		DFS(pre[now][i]);
	}
	tmpPath.pop_back();
}

int main()
{	
	memset(d, 0x3f, sizeof(d));
	memset(c, 0x3f, sizeof(c));
	scanf("%d%d%d%d", &N, &M, &S, &D);
	for (int i = 0; i < M; i++) {
		int c1, c2, distance, cost;
		scanf("%d%d%d%d", &c1, &c2, &distance, &cost);
		d[c1][c2] = distance;
		d[c2][c1] = distance;
		c[c1][c2] = cost;
		c[c2][c1] = cost;
	}
	
	Dijkstra();
	DFS(D);
	
	for (int i = path.size() - 1; i >= 0; i--) {
		printf("%d ", path[i]);
	}
	printf("%d %d\n", dist[D], optValue);
	
	return 0;
}
```

