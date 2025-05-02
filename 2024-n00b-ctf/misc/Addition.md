---
ai_date: '2025-04-27 05:20:02'
ai_summary: Exploited integer overflow in arithmetic calculation to bypass input validation
  and obtain the flag
ai_tags:
- overflow
- integer-overflow
- math
created: 2024-08-04T06:10
description: Just wait 2000 years
points: 100
solves: 501
updated: 2024-08-05T19:03
---

## Cheese method
Since python supports `s[:-1]`, and no checks are done, we can just send -1.

```
how many questions do you want to answer? -1
n00bz{m4th_15nt_4ll_4b0ut_3qu4t10n5}
```

## Hardcore method

```python
from pwn import *
context.log_level = 'error'
s = remote('24.199.110.35', 42189)
s.sendlineafter(b'? ', b'36')
for i in range(36):
    m = s.recvregex(br'what is ([0-9]+) \+ ([0-9]+) =', capture=True)
    s.sendline(str(int(m.group(1)) + int(m.group(2))).encode())
    print(m)
print(s.recvall())
s.close()
```

It will finish in 2179 years.

```flag
n00bz{m4th_15nt_4ll_4b0ut_3qu4t10n5}
```