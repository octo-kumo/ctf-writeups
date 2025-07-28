---
ai_date: 2025-04-27 05:19:27
ai_summary: Binary search vulnerability in the `try_m` function allows for integer overflow exploitation.
ai_tags:
  - overflow
  - binary-search
  - integer-overflow
created: 2024-08-04T06:14
points: 267
solves: 383
updated: 2025-07-14T09:46
---

Simple binary search

```python
from pwn import *
context.log_level = "error"
t = remote("24.199.110.35", 41199)

def try_m(num):
    t.sendline(str(num).encode())
    s = t.recvline().strip()
    if s == b'your answer is too small you silly goose':
        return -1
    elif s == b'your answer is too large you silly goose':
        return 1
    else:
        print(s)
        print(t.recvline())
        exit(0)

def binary_search(l, r):
    while l < r:
        m = (l + r) // 2
        print(m)
        if try_m(m) == 1:
            r = m
        else:
            l = m + 1
    return l

binary_search(0, pow(10, 100))
```

```flag
n00bz{y0u_4r3_4_sm4rt_51l1y_g0053}
```