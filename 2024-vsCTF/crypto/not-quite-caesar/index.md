---
created: 2024-06-14T17:42
updated: 2024-08-04T19:45
solves: 595
points: 141
---

> Caesar??? Couldn't be this!

Just reverse every operation.

```python
import random

out = [354, 112, 297, 119, 306, 369, 111, 108, 333, 110, 112, 92, 111, 315, 104, 102, 285, 102, 303, 100, 112, 94, 111, 285, 97, 351, 113, 98, 108, 118, 109, 119, 98, 94, 51, 56, 159, 50, 53, 153, 100, 144, 98, 51, 53, 303, 99, 52, 49, 128]

random.seed(1337)
flag = []
ops = [
    lambda x: x-3,
    lambda x: x+3,
    lambda x: x//3,
    lambda x: x^3,
]
for v in out:
    flag.append(random.choice(ops)(v))

print("".join(map(chr, flag)))
# vsctf{looks_like_ceasar_but_isnt_a655563a0a62ef74}
```
