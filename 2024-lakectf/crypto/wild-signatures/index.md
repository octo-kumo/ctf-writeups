---
ai_date: '2025-04-27 05:18:52'
ai_summary: Brute-forced the first byte of the signature to bypass validation
ai_tags:
- brute
- padding
- signature
created: 2024-12-08T04:57
points: 109
solves: 93
updated: 2024-12-08T13:17
---

Bruteforce the 1st byte of the signature.

```python
if len(user_msg) > 64 or len(user_msg) == 0:
	print("you're talking too much, or too little")
	exit()
to_verif = user_msg + sig[len(user_msg):]
```

It appears that we need to provide part of the signature, and it can't be a empty string...

Oh well, we can just send a random byte and hope it works `00`.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733651980/2024/12/63d8040e01e839cdfec67f07c52b3ee4.png)

And it does.

## solve

```python
from tqdm import tqdm
from pwn import *
context.log_level = 'error'

for i in tqdm(range(1000)):
    conn = remote('chall.polygl0ts.ch', 9001)
    conn.recvline()
    conn.recvline()
    conn.sendline(b'00')
    res = conn.recvline()
    if res == b'Invalid Ed25519 public key\n' or res == b'The signature is not authentic\n':
        conn.close()
    else:
        print("hooray!")
        print(res)
        conn.sendline(b'00')
        conn.sendline(b'00')
        conn.sendline(b'00')
        conn.sendline(b'00')
        print(conn.recvall())
        conn.close()
        break
```

*Just keep on trying~*

```flag
EPFL{wH4T_d0_yOu_m34n_4_W1LdC4Rd}
```