---
ai_date: '2025-04-27 05:19:41'
ai_summary: 'Reversed shuffle algorithm reveals flag: n00bz{5up3r_dup3r_ultr4_54f3_p455w0rd_4fd1b2d60184}'
ai_tags:
- xss
- reversing
- puzzle
created: 2024-08-04T06:13
points: 451
solves: 177
updated: 2024-08-05T19:02
---

Constant seed random.

```python
from pwn import *
context.log_level = 'error'
s = remote('challs.n00bzunit3d.xyz', 10105)
# every time i send a string, the server replies in the same way
# so i can just reverse the first shuffle
n = '0123456789'
f = '4378052169'  # always got this from the server when sending 0123456789
payload = ''.join([str(f.index(i)) for i in n])
print(payload)
s.sendline(payload.encode())
print(s.recvall().decode())
s.close()
```

```flag
n00bz{5up3r_dup3r_ultr4_54f3_p455w0rd_4fd1b2d60184}
```