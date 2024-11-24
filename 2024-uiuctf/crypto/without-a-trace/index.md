---
created: 2024-06-28T22:58
updated: 2024-11-23T20:31
solves: 298
points: 246
---

We are allowed to change the 5 cells of a 5 by 5 diagonal matrix.

After which the matrix is multiplied with a different 5 by 5 diagonal matrix where each of the 5 values are the long representation of part of the flag.

This means that we are doing $e=a_0f_0+a_1f_0+\dots+a_4f_4$ where $a_n$ is controlled by us, and we have to determine $f_n$.

We can do this by first obtaining $\sum f_n$ with $a_n=1$, then in sequence setting every $a_n$ to 2 and measuring the difference from the base sum.

```python
from pwn import *
context.log_level = 'error'

def inputs(nums):
    conn = remote('without-a-trace.chal.uiuc.tf', 1337, ssl=True)
    conn.recvline()
    conn.recvline()
    for i in nums:
        conn.sendline(str(i))
    conn.recvuntil(b"Have fun: ")
    num = int(conn.recvline().strip())
    conn.close()
    print(nums, "=>", num)
    return num

def solve():
    res = b''
    M1 = inputs([1 for _ in range(5)])
    for i in range(5):
        M2 = inputs([1 if j != i else 2 for j in range(5)])
        res += (M2-M1).to_bytes(5, 'big')
    print(res)

solve()

# [1, 1, 1, 1, 1] => 2000128101369
# [2, 1, 1, 1, 1] => 2504408575853
# [1, 2, 1, 1, 1] => 2440285994541
# [1, 1, 2, 1, 1] => 2426159182680
# [1, 1, 1, 2, 1] => 2163980646766
# [1, 1, 1, 1, 2] => 2465934208374
# b'uiuctf{tr4c1ng_&&_mult5!}'
```
