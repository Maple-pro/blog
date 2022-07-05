---
title: "Codeforces 620 (Div.2)"
date: 2021-03-13 19:30:36
categories: [题解]
mathjax: true
description: Codeforces Round 620, div.2
---

今天被自己写的代码恶心到了……挂出来引以为戒

```c++
#include <bits/stdc++.h>
using namespace std;

const int maxn = 105;
const int maxm = 55;
char a[maxn][maxm];
char result[maxn*maxm];


int main()
{
	int n, m;
	int num = 0;
	int length = 0;
	cin >> n >> m;
	for (int i=1; i<=n; i++)
	{
		scanf("%s", a[i]);
	}

	for (int i=1; i<=n; i++)
	{
		int k;
		for (k=i; k<=n; k++)
		{
			if (a[k][0] != '\0')
			{
				int j;
				for (j=0; j<m; j++)
				{
					if(a[i][j] != a[k][m-j-1])
					{
						break;
					}
				}
				if(j >= m) break;
			}	 
		}

		if (k > n)
		{
			a[i][0] = '\0';
		}
		else if (k == i)
		{
			num++;
		}
		else
		{
			a[k][0] = '\0';
			num = num + 2;
		}
	}

	length = m * num;
	cout << length << endl;

	int r_num = 0;
	for (int i=1; i<=n; i++)
	{
		if(a[i][0] != '\0')
		{
			for (int j=0; j<m; j++)
			{
				result[r_num] = a[i][j];
				r_num++;
			}
		} 
	}

	for (int i=0; i<r_num; i++)
	{
		printf("%c", result[i]);
	}

	for (int i=0; i<r_num; i++)
	{
		printf("%c", result[r_num-i-1]);
	}
	

	return 0;
}
```

## 1304A. Two Rabbits*

没什么好说的，签到题中的签到题，直接上代码。

```c++
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
ll x, y, a, b;

int main()
{
	int t;
	cin >> t;
	while (t--)
	{
		scanf("%I64d %I64d %I64d %I64d", &x, &y, &a, &b);
		if((y-x) % (a+b) == 0)
		{
			ll round = (y-x) / (a+b);
			printf("%I64d\n", round);
		} 
		else
		{
			cout << -1 << endl;
		}
	}
	return 0;
}
```

## 1304B. Longest Palindrome*

其实思路真的挺简单的，就是需要细心。

遍历每个字符串，判断自身是否为palindrome。如果是，将该字符串放在中间；如果不是，那么再遍历后面的字符串，判断是否有一个字符串等于其反串。如果找到了一对，分列两边输出。

注意输出结果中只能有一个自身为palindrome的字符串。同时本题使用`string`，并使用其`reverse()`方法。

还要注意有好的代码习惯，不要一堆`i, j, k`，不然会写得你崩溃。

```c++
#include <bits/stdc++.h>
using namespace std;

const int maxn = 105;
string a[maxn];
string l[maxn];
string r[maxn];
string mid;
bool flag[maxn];
int left_num = 0;
int mid_num = 0;

int main()
{
	memset(flag, true, sizeof(flag));
	int n, m;
	cin >> n >> m;
	for (int i=0; i<n; i++)
	{
		cin >> a[i];
	}

	for (int i=0; i<n; i++)
	{
		if (flag[i] == false) continue;
		string temp = a[i];
		reverse(temp.begin(), temp.end());
		if (temp == a[i])
		{
			mid = a[i];
			if(mid_num == 0) mid_num++;
		}
		else
		{
			for (int j=i+1; j<n; j++)
			{
				if (flag[j] != false && a[j] == temp)
				{
					l[left_num] = a[i];
					r[left_num] = a[j];
					left_num++;
					flag[j] = false;
					break;
				}
			}
		}
		flag[i] = false;
	}

	cout << left_num * 2 * m + mid_num * m << endl;
	for (int i=0; i<left_num; i++) {
		cout << l[i];
	}
	if (mid_num == 1) cout << mid;
	for (int k=0; k<left_num; k++) {
		cout << r[left_num - k - 1];
	}
	cout << endl;

	return 0;
}
```

## 1304C. Air Conditioner*

实际上是一个比较简单的贪心算法。

假设当前时刻温度的最大值为$big$，最小值为$small$。则经过$k$分钟后，温度的最大值为$big+k$，最小值为$small-k$。若此时温度的范围与当前时刻来的客人的温度范围有交集，则说明此时仍可满足条件，否则不满足条件。若满足条件，求取当前时刻温度的最大范围，即$big = min(big, h[i])$，$small = max(small, l[i])$。

```c++
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
const int maxn = 105;
ll t[maxn];
ll l[maxn];
ll h[maxn];

int main()
{
	int q;
	cin >> q;
	while (q--)
	{
		int n;
		ll m;
		cin >> n >> m;
		for (int i=1; i<=n; i++)
			cin >> t[i] >> l[i] >> h[i];

		ll big = m;
		ll small = m;
		ll pre_time = 0;
		bool flag = true;
		for (int i=1; i<=n; i++)
		{
			big = big + t[i] - pre_time;
			small = small - (t[i] - pre_time);
			if(big < l[i] || small > h[i])
			{
				flag = false;
				break;
			}
			pre_time = t[i];
			if (big > h[i]) big = h[i];
			if (small < l[i]) small = l[i]; 
		}

		if (flag == false) cout << "NO" << endl;
		else cout << "YES" << endl;
	}

	return 0;
}
```

## 1304D. Shortest and Longest LIS**

题意即找到从$1$到$n$的两组排列。第一组排列要求其LIS的长度最小，第二组排列要求其LIS的长度最大。并要求得出的排列满足题中所给的相邻两数间的大小关系。

先介绍一下LIS。Longest increasing subsequence (LIS)就是指一个序列的满足下列条件的子序列：序列中的元素为升序，且尽可能长。这个子序列不要求连续，也不要求唯一。

**Shortest LIS**

让序列在整体上看为递减的即可。首先将所有连续递增的部分分组，并设这些组的最大的大小为$m$，则LIS的长度不可能小于$m$。当我们构造这个序列使得LIS的长度等于$m$时，即为所求。

构造方法就是让序列整体上看为递减的即可。

**Longest LIS**

让序列在整体上看为递增的即可。这样所有连续递增的部分可以可以在LIS中，每个连续递减的部分都会有一个元素在LIS中。

题解中还留下了一个有意思的问题，怎么使得LIS的长度等于$k$？

我的思路是首先分类。设所有连续递增部分的长度和为$h$，分为$h<=k$和$h>k$两类情况讨论。具体实现以后再来填坑（挖坑*2）。

```c++
#include <bits/stdc++.h>
using namespace std;

const int maxn = 2*1e5 + 5;
int result[maxn];

int main()
{
	int t;
	cin >> t;
	while (t--)
	{
		memset(result, 0, sizeof(result));
		int n;
		string s;
		cin >> n >> s;

		int pre1 = 0;
		int max_num = n;
		for(int i=0; i<n; i++)
		{
			if(s[i] == '>' || i == n-1)
			{
				for (int j=i; j>=pre1; j--)
				{
					result[j] = max_num;
					max_num--;
				}
				pre1 = i+1;
			}
		}
		for (int i=0; i<n; i++)
			cout << result[i] << " ";
		cout << '\n';

		int pre2 = 0;
		int min_num = 1;
		for (int i=0; i<n; i++)
		{
			if (s[i] == '<' || i == n-1)
			{
				for (int j=i; j>=pre2; j--)
				{
					result[j] = min_num;
					min_num++;
				}
				pre2 = i+1;
			}
		}
		for (int i=0; i<n; i++)
			cout << result[i] << " ";
		cout << '\n';
	}

	return 0;
}
```

## 1304E. 1-Trees and Queries

> 太难了，等我过段时间来补



























