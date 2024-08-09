---
created: 2024-08-08T08:22
updated: 2024-08-08T09:50
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