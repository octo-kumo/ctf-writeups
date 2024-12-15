---
created: 2024-12-14T00:57
updated: 2024-12-15T17:04
points: 775
---

```python
if [usr_hash, pwd] == v:
    if usr == db_user:
        print(f'[+] welcome, {usr} 🤖!')
    else:
        print(f"[+] what?! this was unexpected. shutting down the system :: {open('flag.txt').read()} 👽")
        exit()
    break
```

As we can see this is just MD5 collision stuff.

I searched on google for alphanumeric MD5 hash collision and found this.

[(20) Marc Stevens on X: "Here is a 72-byte alphanum MD5 collision with 1-byte difference for fun: md5("TEXTCOLLBYfGiJUETHQ4hAcKSMd5zYpgqf1YRDhkmxHkhPWptrkoyz28wnI9V0aHeAuaKnak") = md5("TEXTCOLLBYfGiJUETHQ4hEcKSMd5zYpgqf1YRDhkmxHkhPWptrkoyz28wnI9V0aHeAuaKnak")" / X](https://x.com/realhashbreaker/status/1770161965006008570)

So I yoinked it.

```python
import json
from pwn import *
context.log_level = "error"
p1 = "TEXTCOLLBYfGiJUETHQ4hAcKSMd5zYpgqf1YRDhkmxHkhPWptrkoyz28wnI9V0aHeAuaKnak"
p2 = "TEXTCOLLBYfGiJUETHQ4hEcKSMd5zYpgqf1YRDhkmxHkhPWptrkoyz28wnI9V0aHeAuaKnak"

conn = remote("94.237.54.116", 42354)
print(conn.sendlineafter(b":: ", json.dumps({"option": "register"})).decode())
print(conn.sendlineafter(b":: ", json.dumps({"username": p1, "password": "password"})).decode())
print(conn.sendlineafter(b":: ", json.dumps({"option": "register"})).decode())
print(conn.sendlineafter(b":: ", json.dumps({"username": p2, "password": "password"})).decode())
print(conn.sendlineafter(b":: ", json.dumps({"option": "login"})).decode())
print(conn.sendlineafter(b":: ", json.dumps({"username": p2, "password": "password"})).decode())
print(conn.recvall().decode())
```

```flag
HTB{finding_alphanumeric_md5_collisions_for_fun_https://x.com/realhashbreaker/status/1770161965006008570_73cd928afdb3968a7efdc6954fc95bca}
```

Wow turns out the authors found the same twitter article lmao.
