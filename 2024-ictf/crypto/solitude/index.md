---
ai_date: 2025-04-27 05:17:25
ai_summary: Exploited biased random number generator to guess characters in ciphertext
ai_tags:
  - rsa
  - bias
  - frequency-analysis
created: 2024-07-21T16:04
updated: 2025-07-14T09:46
---

I found out that 0 appears very often, and hence can be used to find the true data.

When for example `a` is encoded many times, `a` itself will appear most often in the ciphertext.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721652585/2024/07/7b4991640f79b5d6068f2fe1c8bc155c.png)

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