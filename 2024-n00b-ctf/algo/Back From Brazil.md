---
ai_date: 2025-04-27 05:19:19
ai_summary: Dynamic Programming solution to find the maximum sum path in a grid by comparing top and left neighbors, using 'U' for up and 'L' for left moves. Reversed move sequence gives the flag.
ai_tags:
  - dp
  - grid
  - max-sum
  - dynamic-programming
created: 2024-08-04T06:15
navigation: false
points: 465
solves: 149
updated: 2025-07-14T09:46
---

Find optimal path from top-left to bottom-right which has the greatest sum.

It's a DP problem, we are allowed to move only down and right, and has to reach the sum as given by the challenge.

## summary

1. We create a grid `dp` that represents the maximum possible sum that any endpoint can achieve.
	1. At every point, take the largest of top/left two neighbour tiles and sum it with itself.
	2. Keep track of the choice taken inside `path`.
2. Now start from the bottom right, i.e. the end, and follow the `path` backwards.
3. Reverse the sequence and we have the move sequence.

## explanation

### part 1
Let's first simplify the problem to a 2 by 2 grid.

The only paths possible are via top right or bottom left.

The answer is simple, largest of the two.

### part 2
Now let's assign that final value to the tile $(1,1)$, and expand the grid to 3 by 3.

The tile on the right of $(1,1)$, $(2,1)$ will now compare the value of $(1,1)$ and $(2,0)$, and take the largest of the two to sum with itself.

This cascading sum is the highest possible sum any tile acting as the destination can achieve.

### part 3
Now let's expand the grid to N by N, and for every single tile, do the above of comparing the two neighbors top and left of the tile.

At this point you probably gets it.

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