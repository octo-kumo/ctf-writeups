---
ai_date: 2025-04-27 05:13:48
ai_summary: "ASCII pixel data revealed flag: flag{take_a_break_have_a_snack}"
ai_tags:
  - ascii
  - imgf
  - sub-take
created: 2024-08-08T08:56
points: 200
updated: 2025-07-14T09:46
---

Extracted pixel data are all ~70, and indeed they are just ascii values.

```python
from PIL import Image
im = Image.open('oops-image.png')
pixels = list(im.getdata())
for r, g, b in pixels:
    print(chr(r) + chr(g) + chr(b), end='')
```

```
GVQTMZBXHA3DQNLBGMZTONBTGA2TSNJXG42DMYZVHAZTENBWGY3DKOJWMU2GCNTDGU4TKNZXGQ3DMNRRGQ3TINRTGI2WCNJWGM4TMOBVHAZTGNDFG42TKOJVG42GKNZSGY3DKMJTMQZWI===

5a6d78685a3374305957746c58324666596e4a6c59577466614746325a56396858334e7559574e7266513d3d

ZmxhZ3t0YWtlX2FfYnJlYWtfaGF2ZV9hX3NuYWNrfQ==
```

```flag
flag{take_a_break_have_a_snack}
```