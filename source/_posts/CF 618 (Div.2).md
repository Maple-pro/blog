---
title: "Codeforces 618 (Div.2)"
date: 2021-03-13 19:40:36
categories: [题解]
mathjax: true
description: Codeforces Round 618, div.2
---

## 1300A. Non-zero*

很简单，先判断数组中有多少个零，再判断数组各元素之和+零的个数是否等于零。若等于，输出零的个数+1；若不等于，输出零的个数。

```c++
#include <bits/stdc++.h>
using namespace std;

const int maxn = 100 + 5;
int data[maxn];

int main()
{
	int t;
	cin >> t;
	data[0] = 0;
	while(t--)
	{
		int n;
		int sum = 0;
		int zero_num = 0;
		cin >> n;
		for(int i=1; i<=n; i++)
		{
			cin >> data[i];
			sum += data[i];
			if (data[i] == 0)
			{
				zero_num ++;
			}
		}
		if (sum + zero_num == 0)
		{
			cout << zero_num + 1 << endl;
		}
		else
		{
			cout << zero_num <<endl;
		}
	}

	return 0;
}
```

## 1300B. Assigning to Classes**

当时做的时候猜出来的结论，两个中位数的最小的差值的绝对值为 $a_{n+1}-a_{n}$，下面是证明。

首先对数组排序，$a_1\leq a_2 \leq \ldots \leq a_{2n}$

假设第一个班有$2k+1$个学生，中位数为$a_i$；第二个班有$2l+1$个学生，中位数为$a_j$。且$(2k+1)+(2l+1)=2n \Longrightarrow k+l=n-1$。设$i<j$，即$a_i \leq a_j$。

一班中有$k+1$个同学的能力大于等于$a_i$，二班有$l+1$个同学的能力大于等于$a_j$，由于$a_i \leq a_j$，所以二班中至少有$l+1$个同学的能力大于等于$a_i$。所以至少有$k+l+2=n+1$个同学的能力大于等于$a_i$，因此$a_i \leq a_n$。

同理，至少有$n+1$个学生的能力小于等于$a_j$，因此$a_j \geq a_{n+1} $。

所以，$ |a_j-a_i|\geq a_{n+1}-a_n $。且$a_{n+1}-a_n$可以取到：把$a_n$放到一班，其他所有学生放进二班。

综上，只需要求出数组中第$n$大和第$n+1$大的值即可。

本题只需要把数组sort排序一下就行。

```c++
#include <bits/stdc++.h>
using namespace std;

const int maxn = 1e5 + 5;
int data[2*maxn];

int main()
{
	int t;
	cin >> t;
	while (t--)
	{
		int n;
		cin >> n;
		for (int i=1; i<=2*n; i++){
			scanf("%d", &data[i]);
		}

		sort(data+1, data+n+1);
		int result = data[n+1] - data[n];

		printf("%d\n", result);
	}

	return 0;
}

```

> **拓展**：当然，求第k大还有复杂度更低的方法，比如利用快排中的Partition思想，还有用线段树，最小堆来求。后面再出个专题吧（挖坑中）。敬请期待！

## 1299A. Anu Has a Function**

事实上，$f(a,b)$可以被写为a&$(\sim b)$。

也就是说，数组$[a_1,a_2,\ldots,a_n]$的value值为$ a_1 $ & $(\sim a_2)$ & $\ldots(\sim a_n) $。所以value的值只取决于数组中第一个值。

因此我们可以发现，如果设$n$个数中某一位1的个数为$m$。若$ m \geq 2$，那么value中这一位一定为0；若$m=0$，则value中这一位一定为1。

所以如果某一位1的个数为1，则将这个数放在第一位。为了使value最大，我们从高位往低位搜索。

```c++
#include<bits/stdc++.h>
using namespace std;

typedef long long ll;
const int maxn = 1e5 + 5; //the max size of the array
const int maxpos = 35; //the max position of the number
ll a[maxn];
int num[maxpos];

int main()
{
	memset(num, 0, sizeof(num));
	int n;
	cin >> n;
	for (int i=1; i<=n; i++){
		scanf("%I64d", &a[i]);
	}
	for (int i=1; i<=n; i++){
		for (int j=0; j<=31; j++){
			if (((a[i]>>j) & 1) == 1) num[j]++;
		}
	}

	int now = 35;
	while (now >= 0)
	{
		if (num[now] == 1) break;
		now--;
	}
	int result;
	for (result=1; result<=n; result++)
	{
		if (((a[result]>>now) & 1) == 1) break;
	}

	if (now != -1) {
		printf("%I64d ", a[result]);
	}
	for (int i=1; i<=n; i++){
		if (i != result) {
			printf("%I64d ", a[i]);
		}
	}
	printf("\n");

	return 0;
}
```

## 1299B. Aerodynamic



