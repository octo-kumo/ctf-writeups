---
created: 2024-07-17T00:57
updated: 2024-08-04T19:30
solves: 463
points: 900
---

```python
from pwn import *
import re
context.log_level = 'error'
conn = remote('94.237.51.8', 48661)
conn.recv()

for i in range(999):

    conn.sendline(str(i).encode())
    t = conn.recvuntil(b'Enter an index: ').decode()
    t = re.search('at Index \\d+: (.)\n', t).group(1)
    print(t, end='', flush=True)
    if t == '}':
        break
print()
conn.close()
```
