---
title: "Edu Codeforces R 90 (Div.2)"
date: 2021-03-13 19:20:36
categories: [题解]
mathjax: true
description: Educational Codeforces Round 90, div.2
---

## A. Donut Shops*

两个临界条件：

买一个时如果甲店更便宜，$ x1=1 $，否则，$ x1=-1 $；

买$ b $件时如果乙店更便宜，$ x2=b $，否则，$ x2=-1 $。

```c++
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;

int main()
{
	int t;
	cin >> t;
	while (t--)
	{
		ll a, b, c;
		cin >> a >> b >> c;
		ll x1 = -1;
		ll x2 = -1;
		if (a < c)
		{
			x1 = 1;
		}
		if (a * b > c)
		{
			x2 = b;
		}
		cout << x1 << " " << x2 << endl;
	}
	return 0;
}
```

## B. 01Game*

从最终结果来看，要么最后只剩下1，要么最后只剩下0，因此只用找到0和1个数的最小值，然后判断奇偶性即可。

```c++
#include <bits/stdc++.h>
using namespace std;

const int maxn = 105;

int main()
{
	int t;
	cin >> t;
	while (t--)
	{
		char s[maxn];
		cin >> s;
		int len = strlen(s);
		int n1 = 0;
		int n2 = 0;
		int minn = 0;
		for (int i = 0; i< len; i++)
		{
			if (s[i] == '1')
			{
				n1++;
			}
			else
			{
				n2++;
			}
		}
		if (n1 <= n2)
		{
			minn = n1;
		}
		else
		{
			minn = n2;
		}
		if (minn % 2 == 0)
		{
			cout << "NET" << endl;
		}
		else
		{
			cout << "DA" << endl;
		}
	}
	return 0;
	
}
```

## C. Pluses and Minuses**

重点是理解题意。从左到右遍历字符串，当$ num(-)-num(+)>cur $时，$ cur=cur+1 $，同时从头开始再次遍历。输出为遍历的字符的个数。

我们采用动态规划算法，算法复杂度为O(n)。从左到右遍历字符串，当$ num(-)-num(+)>cur $时，$ cur=num(-)-num(+) $，记下此时的结果，$ ans+=i+1 $。然后不用重头开始，可以继续遍历。

```c++
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;

int main()
{
	int t;
	ll ans;
	cin >> t;
	while (t--)
	{
		string s;
		cin >> s;
		ans = s.length();
		ll sum = 0;
		ll res = 0;
		for (int i = 0; i < s.length(); i++)
		{
			if (s[i] == '+') sum++;
			else sum--;
			if (sum < res)
			{
				res = sum;
				ans += i + 1;
			}
		}
		cout << ans << endl;
	}
	return 0;
}
```

## D. Maximum Sum on Even Positions

To be continue...