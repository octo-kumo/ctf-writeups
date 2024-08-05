---
created: 2024-06-15T06:42
updated: 2024-08-04T19:45
solves: 23
points: 490
---

I was able to get a string `__import__` but that's it.

```python
from pwn import *
import string

context.log_level='critical'
# Define the target server details
server = "vsc.tf"
port = 6001

# Define the set of characters to try
charset = string.ascii_letters + string.digits

# Connect to the server
conn = remote(server, port)
conn.recvuntil(b"Enter code: ")
payload = '''
function_name = chr(95) + chr(95) + "import" + chr(95) + chr(95)
print(function_name)
'''

conn.sendline(payload.encode()+b"\n")
conn.sendline(b"#EOF\n")
print(conn.recvall().decode())
exit()
```

Idk.
