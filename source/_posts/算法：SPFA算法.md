---
title: 算法：SPFA算法
date: 2021-04-14 19:06:09
tags: [图, 最短路]
categories: [算法]
description: SPFA的板子
mathjax: true
---

SPFA (Shortest Path Faster Algorithm)

期望时间复杂度为 $O(kE)$，其中 E 是图的边数，k 是一个常数，一般 k 不超过 2.

如果图中有从源点可达的负环，传统SPFA的时间复杂度就会退化为 $O(VE)$.

SPFA算法是从Bellman-Ford算法优化而来。

> 优化点：只有当某个顶点 u 的 $d[u]$ 值改变时，从它出发的边的临界点 v 的 $d[v]$ 值才有可能被改变

伪代码：

```c++
queue<int> Q;
源点s入队;
while (队列非空) {
    取出队首元素u;
    for (u的所有邻接边 u->v) {
        if (d[u] + dis > d[v]) {
            d[v] = d[u] + dis;
            if (v当前不在队列) {
                v入队;
                if (v入队次数大于n-1) {
                    return false; // 说明有可达负环
                }
            }
        }
    }
    return true; // 无可达负环
}
```

代码模板：

```c++
struct Node {
	int v; // 邻接边的目标顶点 
	int dis; // 邻接边的边权
	Node(int _v, int _dis): v(_v), dis(_dis) {} 
};

const int INF = 0x3f3f3f3f;
vector<Node> Adj[MAXV]; // 邻接表 
int n, d[MAXV], num[MAXV]; // num数组记录顶点的入队次数
bool inq[MAXV]; // 顶点是否在队列中 

bool SPFA(int s) {
	memset(inq, false, sizeof(inq));
	memset(num, 0, sizeof(num));
	memset(d, 0x3f, sizeof(d));
	
	queue<int> Q;
	Q.push(s);
	inq[s] = true;
	num[s]++;
	d[s] = 0;
	while (!Q.empty()) {
		int u = Q.front();
		Q.pop();
		inq[u] = false;
		for (int i = 0; i < Adj[u].size(); i++) {
			int v = Adj[u][i].v;
			int dis = Adj[u][i].dis;
			// 松弛操作
			if (d[u] + dis < d[v]) {
				d[v] = d[u] + dis;
				if (!inq[v]) {
					Q.push(v);
					inq[v] = true;
					num[v]++;
					if (num[v] >= n) return false; // 有可达负环 
				}
			} 
		} 
	}
	
	return true; // 无可达负环 
}
```



