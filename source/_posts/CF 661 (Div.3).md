---
title: "Codeforces 661 (Div.3)"
date: 2021-03-13 19:22:36
categories: [题解]
mathjax: true
description: Codeforces Round 661, div.3
---

> 我胡汉三又回来啦，这段时间会更新题解和MIT6.828的内容，可能会更其他的学习笔记什么的

## A. Remove Smallest*

贪心，排序。

签到题。

把数据排序一下，然后遍历即可。

```c++
/**
 * @author: Maples
 * @create: 2020/08/05
 * @contest: CF #661
 * @problem: A
 * @description: 排序
 * @difficulty: *
 */

#include <bits/stdc++.h>
using namespace std;

const int maxn = 55;
int a[maxn];

int main()
{
    int t;
    cin >> t;
    while (t--)
    {
        memset(a, 0, sizeof(a));
        int n;
        cin >> n;

        for (int i = 0; i < n; i++)
        {
            cin >> a[i];
        }

        if (n == 1)
        {
            cout << "YES" << endl;
            continue;
        }

        sort(a, a + n);
        bool flag = true;
        for (int i = 0; i < n-1; i++)
        {
            if ((a[i + 1] - a[i]) > 1)
            {
                flag = false;
                break;
            }
        }

        if (flag) cout << "YES" << endl;
        else cout << "NO" << endl;
    }
    // system("pause");
    return 0;
}
```

## B. Gifts Fixing*

贪心。

签到题。

首先获取$ a_i $的最小值$ min(a) $和$ b_i $的最小值$ min(b) $。然后对于每一份礼物，最小步数为$ max(a_i-min(a), b_i-min(b)) $。因此，答案为$ \sum_{i=1}^{n} {max(a_i-min(a), b_i-min(b))}$。

```c++
/**
 * @author: Maples
 * @create: 2020/08/05
 * @contest: CF #661
 * @problem: B
 * @description: 贪心
 * @difficulty: *
 */

#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
const int maxn = 55;
ll a[maxn];
ll b[maxn];

int main()
{
    int t;
    cin >> t;
    while (t--)
    {
        ll result = 0;
        memset(a, 0, sizeof(a));
        memset(b, 0, sizeof(b));
        int n;
        cin >> n;
        for (int i = 0; i < n; i++)
        {
            cin >> a[i];
        }
        for (int i = 0; i < n; i++)
        {
            cin >> b[i];
        }

        int minA = *min_element(a, a + n);
        int minB = *min_element(b, b + n);

        for (int i = 0; i < n; i++)
        {
            int x = a[i] - minA;
            int y = b[i] - minB;
            int z = std::min(x, y);
            result += x + y - z;
        }
        cout << result << endl;
    }

    // system("pause");
    return 0;
}
```

