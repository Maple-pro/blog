---
title: 武汉大学新生寒假集训---Day5
date: 2021-03-13 19:16:36
categories: [题解]
mathjax: true
description: Editorial for training in WHU, day 5.
---

> 开始写题解了，一方面做一个记录，一方面督促自己。奥里给！
> 题号后的 * 为难度系数

## A. Solving Order*
[HDU - 5702](http://acm.hdu.edu.cn/showproblem.php?pid=5702)
结构体排序

```cpp
#include <bits/stdc++.h>
using namespace std;

struct problem{
	string color;
	int num;
};
const int maxn = 10;

int comp(const problem &p1, const problem &p2)
{
	return p1.num > p2.num;
}

int main()
{
	int T;
	cin >> T;
	while (T--)
	{
		problem p[10];
		int n;
		cin >> n;
		for (int i=0; i<n; i++)
		{
			cin >> p[i].color >> p[i].num;
		}

		sort (p, p+n, comp);

		for(int i=0; i<n-1; i++)
		{
			cout << p[i].color << ' ';
		}
		cout << p[n-1].color;
		cout << endl;
	}

	return 0;
}
```

## B. Luck Compition*
[HDU - 5704](http://acm.hdu.edu.cn/showproblem.php?pid=5704)
公式推导

```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 100 + 5;

int main()
{
	int T;
	cin >> T;
	while (T--)
	{
		int N;
		cin >> N;
		int summ = 0;
		int num[maxn];
		for(int i=0; i<N-1; i++)
		{
			cin >> num[i];
			summ = summ + num [i];		
		}

		double stand = 2 * (double)summ / (double)(3 * N -2);

		int i;    //最后的结果
		for(i=0; i<=100; i++)
		{
			if (i > stand)
			{
				i--;
				break;
			}
		}

		int a = 1;
		for(int j=0; j<N; j++)
		{
			if(num[j] == i) a++;
		}

		double per;
		if(a==0) per = 1.00;
		else per = 1/(double)a;

		cout.setf(ios::fixed);
		cout << i << ' ' << fixed << setprecision(2) << per <<endl;
	}

	return 0;
}

```

## F. N! Again*
[HDU - 2674](http://acm.hdu.edu.cn/showproblem.php?pid=2674)
暑假集训的时候还做过，还是我想出来的，怎么就忘了呢……
因为$ 2019 = 7 * 7 * 41 $
所以$ N \geq 41 $时，结果都为0
因此只用考虑$ N < 41 $的情况

```cpp
#include<bits/stdc++.h>
using namespace std;

int main()
{
	long long N;
	long long ans;
	while (scanf("%lld", &N) != EOF)
	{
		if (N >= 41)
		{
			printf("0\n");
		}
		else if (N == 0)
		{
			printf("1\n");
		}
		else
		{
			ans = 1;
			for (int i=1; i<=N; i++)
			{
				ans = (ans * i) % 2009;
			}
			printf("%lld\n", ans);
		}
	}

	return 0;
}
```

## G. god is a girl*
[HDU - 2672](http://acm.hdu.edu.cn/showproblem.php?pid=2672)
找规律，斐波那契数列
每个字母项加上对应的斐波那契数列的值，非字母项不变
卡在了终止循环的条件上……看来字符串类型的输入还是要整明白
还有就是斐波那契数列要转换一下，不然会爆`long long `

```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 1e3 + 2;
char sen[maxn];
long long fib[maxn];

int main()
{
	fib[0] = 1; fib[1] = 1;
	for(int i=2; i<maxn; i++)
	{
		fib[i] = (fib[i-1] + fib[i-2]) % 26;
	}


	while(gets(sen) != NULL)
	{
		int len = strlen(sen);
		int count = 0;

		for(int i=0; i<len; i++)
		{
			if (sen[i] >= 'A' && sen[i] <='Z')
			{
				sen[i] = 'A' - 1 + (sen[i] + fib[count] - 'A' + 1) % 26;
				count ++;
			}
		}
		cout << sen << endl;
	}

	return 0;
}
```