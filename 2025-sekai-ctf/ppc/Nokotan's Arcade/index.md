---
ai_date: 2025-08-17 20:46:47
ai_summary: Solved a dynamic programming problem involving scheduling players with
  constraints based on start and end times.
ai_tags:
- dp
- scheduling
- time-constraints
created: 2025-08-16T11:24
points: 100
solves: 392
title: Nokotan's Arcade
updated: 2025-08-17T20:46
---

> GPT solved it

Nice problem — it's a 1-machine scheduling/DP problem where each possible start minute can pick the best available player at that minute.
Here's a clean, fast solution (O((n+m) log m) time, O(n+m) memory).

## Idea

- A game started at integer minute `s` occupies `[s, s+t-1]`. Player `i` can start at any `s` with `li ≤ s ≤ ri - t + 1`. If `ri - t + 1 < li` the player can't play at all.
- For each start minute `s` we only ever need the **maximum** `pi` among players that allow start at `s` (because choosing a player doesn't constrain future choices).
- Let `best[s]` be that maximum (0 if none). Then DP:

  ```
  dp[pos] = max(dp[pos+1], best[pos] + dp[pos+t])
  ```

  (skip the minute, or start a game at pos and jump by t).

- Compute `best` by sweeping `pos = 1..n`, maintaining active players using a max-heap and lazy removals (add at `li`, remove at `ri-t+1 + 1`).

## Complexity

- Building events: O(m)
- Sweep with heap: O((n + m) log m)
- DP: O(n)
- Works for constraints up to 1e5.

```python
import sys
import heapq

def solve():
    data = list(map(int, sys.stdin.buffer.read().split()))
    it = iter(data)
    n = next(it); m = next(it); t = next(it)
    start_events = [[] for _ in range(n + 3)]
    end_events   = [[] for _ in range(n + 3)]
    pis = [0] * m

    for i in range(m):
        l = next(it); r = next(it); p = next(it)
        pis[i] = p
        b = r - t + 1
        if b >= l:
            start_events[l].append(i)      # add player i at minute l
            if b + 1 <= n + 1:
                end_events[b + 1].append(i)  # remove player i at minute b+1

    active = [False] * m
    heap = []  # store (-p, idx)
    best = [0] * (n + 2)

    for pos in range(1, n + 1):
        # process removals (players whose valid range ended before pos)
        for idx in end_events[pos]:
            active[idx] = False
        # process additions beginning at pos
        for idx in start_events[pos]:
            active[idx] = True
            heapq.heappush(heap, (-pis[idx], idx))
        # clean up heap top if it's inactive
        while heap and not active[heap[0][1]]:
            heapq.heappop(heap)
        if heap:
            best[pos] = -heap[0][0]
        else:
            best[pos] = 0

    # DP from n down to 1
    dp = [0] * (n + t + 5)  # safe padding
    for pos in range(n, 0, -1):
        dp[pos] = max(dp[pos + 1], best[pos] + dp[pos + t])

    print(dp[1])

if __name__ == "__main__":
    solve()
```

```flag
SEKAI{https://youtu.be/-PTe8zkYt9A}
```