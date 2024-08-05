---
created: 2024-08-04T06:14
updated: 2024-08-05T19:05
points: 403
solves: 248
---

Just bunch of numbers

Simple stuff, testing just pwntool/socket familiarity.

```python
import math
import re
from pwn import *
context.log_level = "error"
t = remote("challs.n00bzunit3d.xyz", 10349)

def maxPrimeFactor(n):
    maxPrime = -1
    while n % 2 == 0:
        maxPrime = 2
        n >>= 1
    for i in range(3, int(math.sqrt(n)) + 1, 2):
        while n % i == 0:
            maxPrime = i
            n = n / i
    if n > 2:
        maxPrime = n
    return int(maxPrime)

r_lcm = rb'^Give me the least common multiple of (\d+) and (\d+): $'
r_gcd = rb'^Give me the greatest common divisor of (\d+) and (\d+): $'
r_gpf = rb'^Give me the greatest prime factor of (\d+): $'

while True:
    r = t.recvregex(rb"Current round: (\d+) of 100\n", capture=True).group(1).decode()
    print("round", r)
    l = t.recv()
    print(l)
    if _m := re.match(r_lcm, l):
        t.sendline(str(math.lcm(int(_m.group(1)), int(_m.group(2)))).encode())
    if _m := re.match(r_gcd, l):
        t.sendline(str(math.gcd(int(_m.group(1)), int(_m.group(2)))).encode())
    if _m := re.match(r_gpf, l):
        t.sendline(str(maxPrimeFactor(int(_m.group(1)))).encode())
    print(t.recvline())
    if r == "100":
        break
print(t.recvline())
print(t.recvline())
```

```flag
n00bz{numb3r5_4r3_fun_7f3d4a_ff299351f77f
```
