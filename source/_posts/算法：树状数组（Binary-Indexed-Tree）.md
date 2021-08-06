---
title: 算法：树状数组/二叉索引树（Binary Indexed Tree）
date: 2021-03-15 19:58:36
tags: [树状数组, 二叉索引树]
categories: [算法]
description: 介绍树状数组的原理、代码实现和相应例题。参考《算法竞赛入门经典 训练指南》
mathjax: true
---

# 树状数组

## 1. 原理

>  树状数组可以高效地求出连续的一段元素之和或者更新单个元素的值。但是无法高效地给某一个区间的所有元素同时加上一个值。

支持两种操作：

- `Add (x, d)`：让$ A_x $增加$ d $
- `Query (R)`：以$ O(log n) $的复杂度计算前缀和

![image-20210315200852481](https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210315200852481.png)

![image-20210315200956732](https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210315200956732.png)

![image-20210315201046181](https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210315201046181.png)

注意：

array indices的为$ x $，则$ C[x] $表示的是$ [x-lowbit(x), x] $的个数总和。



## 2. 代码实现

三个操作的代码如下：

```c++
int lowbit (int x) {
    return x & -x;
}
```

```c++
int sum (int x) {
    int ret = 0;
    while (x > 0) {
        ret += C[x];
        x -= lowbit(x);
    }
}
```

```c++
void add (int x, int d) {
    while (x <= n) {
        C[x] +=d;
        x += lowbit(x);
    }
}
```



## 3. 例题

> [乒乓比赛](https://vjudge.net/problem/UVALive-4329)

考虑第$ i $个人当裁判的情形：

此时裁判的技能值为$ a_i $，假设$a_1$到$ a_{i-1} $中有$ c_i $个比$ a_i $小，则有$ (i-1)-c_i $个比$ a_i $大；假设$ a_{i+1} $到$ a_n $中有$ d_i $个比$ a_i $小，则有$ (n-i)-d_i $个比$ a_i $大。则组织比赛的数量为$ \sum_{i=1}^{n}{c_i*(n-i-d_i) + (i-1-c_i)*d_i} $。因此问题转化为求解当第$ i $个人当裁判时，$ c_i $和$ d_i $的值。

$ c_i = x(1) + x(2) + ... + x(a_i-1) = sum(a_i-1) $，$ x[i] $表示$ i $出现的个数，也就是说我们需要动态地修改单个元素值并且求前缀和，使用BIT算法。

求$ c_i $时，从左至右遍历，边修改边求前缀和；求$ d_i $时，从右至左遍历，边修改边求前缀和。

```c++
// 树状数组 
#include <bits/stdc++.h>
using namespace std;

const int maxn = 2e4 + 5;
const int maxa = 1e5 + 5;
int a[maxn];
int left_bit[maxa]; // left_bit
int right_bit[maxa]; // right_bit
int c[maxn]; // left
int d[maxn]; // right
int n;

int lowbit(int x)
{
	return x & -x;
}

int sum1(int x)
{
	int ret = 0;
	while (x >= 1) {
		ret += left_bit[x];
		x -= lowbit(x);
	}
	return ret;
}

int sum2(int x)
{
	int ret = 0;
	while (x >= 1) {
		ret += right_bit[x];
		x -= lowbit(x);
	}
	return ret;
}

void add1(int x)
{
	while (x <= maxa) {
		left_bit[x]++;
//		printf("%d %d\n", x, left_bit[x]);
		x += lowbit(x);
	}
}

void add2(int x)
{
	while (x <= maxa) {
		right_bit[x]++;
		x += lowbit(x);
	}
}

int main()
{
	int t; cin >> t;
	while (t--) {
		memset(a, 0, sizeof(a));
		memset(left_bit, 0, sizeof(left_bit));
		memset(right_bit, 0, sizeof(right_bit));
		memset(c, 0, sizeof(c));
		memset(d, 0, sizeof(d));
		long long res = 0;
		cin >> n;
		for (int i = 1; i <= n; i++) cin >> a[i];
		for (int i = 1; i <= n; i++) {
			c[i] = sum1(a[i] - 1);
//			printf("a[i] is: %d\n", a[i]);
			add1(a[i]);
		}
		for (int i = n; i >= 1; i--) {
			d[i] = sum2(a[i] - 1);
			add2(a[i]);
		}
		for (int i = 2; i <= n-1; i++) {
			res += c[i] * (n - i - d[i]) + d[i] * (i - 1 - c[i]);
		}
//		for (int i = 1; i <= n; i++) {
//			printf("%d %d\n", c[i], d[i]);
//		}
		cout << res << endl;
	}
}
```



# Reference

- [1] 《算法竞赛入门经典 训练指南》