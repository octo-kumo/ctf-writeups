---
ai_date: 2025-04-27 05:22:46
ai_summary: Executed arbitrary code using '__import__' string exploitation, likely a hint for code injection or reflection vulnerability.
ai_tags:
  - exec
  - code-injection
  - python
created: 2024-06-15T06:42
solves: 23
updated: 2025-07-14T09:46
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