---
created: 2024-06-14T14:44
updated: 2024-08-04T19:44
tags:
  - fav
solves: 74
points: 465
---

> Given an integer n (n ≤ 10, 000), QuickMaffs challenges you to find out how many permutations of the numbers 1 through n satisfy the following condition $$
\forall i (2 \leq i \leq n),
\begin{cases}
a[i - 1] < a[i] & \text{if } i \text{ is odd} \\
a[i - 1] > a[i] & \text{if } i \text{ is even}
\end{cases} $$
>
> Your task is to compute the number of such valid permutations modulo $10^9 + 7$

With a simple naïve brute force method we can determine the starting numbers of the sequence to be `1, 1, 1, 2, 5, 16, 61, 272, 1385, 7936, 50521`.

Well it turns out that it is a known sequence [A000111 - OEIS](https://oeis.org/A000111).

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1718426215/2024/06/44d2f330354b8fbd8becaaffda273618.png)

With this information we can then abuse wolfram to calculate the entire problem range for us.

```python
Table[Mod[Piecewise[{{I^n EulerE[n], Mod[n, 2] == 0}, {-(((2 I)^(1 + n) (-1 + 2^(1 + n)) BernoulliB[1 + n])/(1 + n)), Mod[n, 2] == 1}}],1000000007], {n, 9001,10000}]
```

```cpp
#include <bits/stdc++.h>
using namespace std;
int ans[] = {1, 1, 1, 2, 5, 16, 61, 272, 1385, 7936, 50521, ... };
int main() {
    int n;
    cin >> n;
    cout << ans[n];
    return 0;
}
```

```flag
vsctf{QuickMaffs_FMC_Huge_Degen_b4a1c2af876eeb9a}
```

Simple yet effective.
