---
created: 2024-08-04T06:10
updated: 2024-08-05T19:04
points: 448
solves: 182
---

When a set of numbers $x\in[0,n]$ are taken abs difference with the middle number $n/2$, we are effectively halving the size of that range. ($[0,n/2]$)
Furthermore, since $2^20=1048576>696969$, we know for certain this will work.

```python
from pwn import *
ns = []
N = 696969
for i in range(20):
    ns.append(N//2+1)
    N //= 2
s = remote('challs.n00bzunit3d.xyz', 10051)
for n in ns:
    s.sendline(str(n).encode())
print(s.recvall())
```

```flag
n00bz{1_sh0uld_t34ch_my_br0th3r_logs_4ab908d69174}
```
