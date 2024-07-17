---
created: 2024-07-17T00:57
updated: 2024-07-17T01:12
---
## Solve Script

Watched some random news channel on world news and afk-ed the script.


```python
# nc 94.237.51.8 42152
from pwn import *
import re
context.log_level = 'error'
conn = remote('94.237.51.8', 42152)
conn.recvuntil(b'(y/n) ')
conn.sendline(b'y')

'''
If I tell you there's a GORGE, you send back STOP
If I tell you there's a PHREAK, you send back DROP
If I tell you there's a FIRE, you send back ROLL
'''
M = {
    'GORGE': 'STOP',
    'PHREAK': 'DROP',
    'FIRE': 'ROLL'
}


def go():
    text = conn.recvuntil(b'What do you do?').decode()
    print(text)
    res = re.findall(r'(GORGE|PHREAK|FIRE)', text)
    res = [M[c] for c in res]
    conn.sendline('-'.join(res).encode())


try:
    while True:
        go()
except EOFError:
    print(conn.recvall())
finally:
    conn.close()
# HTB{1_wiLl_sT0p_dR0p_4nD_r0Ll_mY_w4Y_oUt!}
```