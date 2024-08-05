---
created: 2024-08-04T06:10
updated: 2024-08-04T21:18
description: Just wait 2000 years
points: 100
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
