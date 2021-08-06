---
title: "Codeforces 652 (Div.2)"
date: 2021-03-13 19:24:36
categories: [题解]
mathjax: true
description: Codeforces Round 652, div.2
---

## A. FashionanLee*

签到题

利用凸多边形外角和为360°可得，当$ 90 \pmod{360/n} = 0 $ 时，满足要求

```c++
#include <bits/stdc++.h>
using namespace std;

int main()
{
	int t;
	cin >> t;
	while (t--)
	{
		int n;
		cin >> n;
		if (n % 4 == 0)
		{
			cout << "YES" << endl;
		}
		else
		{
			cout << "NO" << endl;
		}
	}
	return 0;
}
```

## B. AccurateLee*

从左到右找第一个1，记下位置为flag1，如果整个序列都没有1，$ flag1=-1$；

从右到左找第一个0，记下位置为flag2，如果整个序列都没有0，$ flag2=-1$。

当$ flag1=-1 \bigvee flag2=-1 \bigvee flag1 > flag2 $，输出整个序列；

否则，输出flag1左边的序列（不包含flag1）和flag2右边的序列（包含flag2）。

```c++
#include<bits/stdc++.h>
using namespace std;

const int maxn = 1e5 + 5;
char s[maxn];

int main()
{
	int t;
	cin >> t;
	while (t--)
	{
		int n;
		cin >> n;
		int flag1 = -1;
		int flag2 = -1;
		memset(s, 0, sizeof(s));
		cin >> s;

		for (int i = 0; i < n; i++)
		{
			if (s[i] == '1')
			{
				flag1 = i;
				break;
			}
		}
		for (int i = n - 1; i >= 0; i--)
		{
			if (s[i] == '0')
			{
				flag2 = i;
				break;
			}
		}

		if (flag1 == -1 || flag2 == -1)
		{
			for (int i = 0; i < n; i++)
			{
				cout << s[i];
			}
			cout << endl;
		}
		else if (flag1 < flag2)
		{
			for (int i = 0; i < flag1; i++)
			{
				cout << s[i];
			}
			for (int i = flag2; i < n; i++)
			{
				cout << s[i];
			}
			cout << endl;
		}
		else
		{
			for (int i = 0; i < n; i++)
			{
				cout << s[i];
			}
			cout << endl;
		}
	}
	return 0;
}
```

## C. RationalLee

To be continue...