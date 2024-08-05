---
created: 2024-08-04T06:15
updated: 2024-08-04T21:17
points: 465
---

Find optimal path from top-left to bottom-right which has the greatest sum.

It's a DP problem, we are allowed to move only down and right, and has to reach the sum as given by the challenge.

1. We create a grid `dp` that represents the maximum possible sum that any endpoint can achieve.
	1. At every point, take the largest of top/left two neighbour tiles and sum it with itself.
	2. Keep track of the choice taken inside `path`.
2. Now start from the bottom right, i.e. the end, and follow the `path` backwards.
3. Reverse the sequence and we have the move sequence.
## solve script

```python
from pwn import *
context.log_level = "error"

def max_path(grid):
    m, n = len(grid), len(grid[0])
    dp = [[0] * n for _ in range(m)]
    path = [[''] * n for _ in range(m)]
    dp[0][0] = grid[0][0]
    for j in range(1, n):
        dp[0][j] = dp[0][j-1] + grid[0][j]
        path[0][j] = 'L'
    for i in range(1, m):
        dp[i][0] = dp[i-1][0] + grid[i][0]
        path[i][0] = 'U'
    for i in range(1, m):
        for j in range(1, n):
            if dp[i-1][j] > dp[i][j-1]:
                dp[i][j] = grid[i][j] + dp[i-1][j]
                path[i][j] = 'U'
            else:
                dp[i][j] = grid[i][j] + dp[i][j-1]
                path[i][j] = 'L'
    i, j = m-1, n-1
    move_sequence = []
    while i > 0 or j > 0:
        if path[i][j] == 'U':
            move_sequence.append('d')
            i -= 1
        else:
            move_sequence.append('r')
            j -= 1
    move_sequence.reverse()
    return dp[m-1][n-1], ''.join(move_sequence)

t = remote("24.199.110.35", 43298)
n = 1000
for i in range(10):
    print("round", i)
    grid = []
    for i in range(n):
        l = [int(x) for x in t.recvline().strip().split()]
        grid.append(l)
    oans = int(t.recvregex(rb'optimal: (\d+) \xf0\x9f\xa5\x9a\n', capture=True).group(1).decode())
    mans, path = max_path(grid)
    if oans != mans:
        print("Wrong answer")
        exit()
    t.sendline(path.encode())
print(t.recvall().decode())
```

```flag
n00bz{1_g0t_b4ck_h0m3!!!}
```
