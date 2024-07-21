---
created: 2024-07-21T16:04
updated: 2024-07-21T18:47
---

I found out that 0 appears very often, and hence can be used to find the true data.

But I did not think about dividing the output into columns, and analyze each character individually...

```python
from pwn import *
import pandas as pd
from collections import Counter
context.log_level = 'error'
conn = remote("solitude.chal.imaginaryctf.org", 1337)
conn.recvuntil(b"got flag? ")
conn.sendline(b"10000")
data = conn.recvuntil(b"got flag? ", drop=True)
data = [bytes.fromhex(x.decode()) for x in data.split(b"\n") if x]
data = [[data[i][j] for i in range(len(data))] for j in range(len(data[0]))]
for c in data:
    print(chr(Counter(c).most_common(1)[0][0]), end="")
conn.close()
```

```flag
ictf{biased_rng_so_sad_6b065f93}
```
