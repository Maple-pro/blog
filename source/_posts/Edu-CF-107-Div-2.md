---
title: Educational Codeforces 107 (Div.2)
date: 2021-04-13 23:04:31
tags: [贪心, 构造算法, 大模拟]
categories: [题解]
mathjax: true
description: Educational Codeforces Round 107, div.2
---

## A. Review Site

贪心。把所有1号3号投一边，2号投一边

```c++
#include <bits/stdc++.h>
using namespace std;

int main()
{	
	int t;
	scanf("%d", &t);
	while (t--) {
		int n;
		int sum = 0;
		scanf("%d", &n);
		for (int i = 0; i < n; i++) {
			int r;
			scanf("%d", &r);
			if (r == 1 || r == 3) sum++;
		}
		printf("%d\n", sum);
	}
}
```

## B. GCD Length

构造算法。巧妙的构造算法

```c++
// constructive algorithms
#include <bits/stdc++.h>
using namespace std;

int main()
{	
	int t;
	scanf("%d", &t);
	while (t--) {
		int a, b, c;
		scanf("%d%d%d", &a, &b, &c);
		for (int i = 1; i <= a - c + 1; i++) printf("1");
		for (int i = 1; i <= c - 1; i++) printf("0");
		printf(" ");
		printf("1");
		for (int i = 1; i <= b - 1; i++) printf("0");
		printf("\n");
	}
	
	return 0;
}
```

## C. Yet Another Card Deck

大模拟。实际上只有第一次出现（序号最小的）的卡片才有用，之后出现的卡片没有用，因此只需要记录不同颜色的卡片第一次出现的位置即可。

```c++
#include <bits/stdc++.h>
using namespace std;

const int maxn = 3e5 + 5;
const int maxa = 55;
int n, q;
int a[maxn]; // 记录源数据 
int p[maxa]; // 记录每个颜色出现位置的最小值 

int main()
{	
	scanf("%d%d", &n, &q);
	memset(p, -1, sizeof(p));
	for (int i = 1; i <= n; i++) scanf("%d", &a[i]);
	for (int i = n; i >= 1; i--) p[a[i]] = i;
	
	for (int i = 0; i < q; i++) {
		int query;
		scanf("%d", &query);
		printf("%d", p[query]);
		if (i < q - 1) printf(" ");
		else printf("\n");
		for (int j = 1; j <= maxa; j++) {
			if (p[j] != -1 && p[j] < p[query]) p[j]++;
		}
		p[query] = 1;
	}
	
	return 0;
}
```

