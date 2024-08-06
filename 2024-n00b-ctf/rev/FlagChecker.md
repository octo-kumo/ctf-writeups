---
created: 2024-08-04T06:07
updated: 2024-08-05T19:42
points: 431
solves: 210
tags:
  - vba
---

`xlsm`'s `m` stands for macro. This excel file has macros in it.

```bash
pcodedmp .\FlagChecker.xlsm -o FlagChecker.dump
```

We can observe that 24 characters are loaded and checked in the p-code.

After some translation... we have our z3 solver.

```python
from z3 import *

chrs = [BitVec(f"char_{i}", 8) for i in range(25)]

s = Solver()

for i in range(25):
    s.add(chrs[i] >= 0x20)
    s.add(chrs[i] <= 0x7E)

s.add(chrs[1] ^ chrs[8] == 0x0016)  # Line #31:
s.add(chrs[10] + chrs[24] == 0x00B0)  # Line #32:
s.add(chrs[22] - chrs[9] == 0x0009)  # Line #33:
s.add(chrs[22] ^ chrs[6] == 0x0017)  # Line #34:
s.add((chrs[12] / 0x0005) * (chrs[12] / 0x0005)*(chrs[12] / 0x0005)*(chrs[12] / 0x0005) == 0xFD11)

s.add(chrs[22] == chrs[11])
s.add(chrs[15] * chrs[8] == 0x36D8)
s.add(chrs[12] ^ (chrs[17]-0x0005) == 0x0005)

s.add(chrs[18] == chrs[23])
s.add(chrs[13] ^ chrs[14] ^ chrs[2] == 0x0079)

s.add(chrs[14] ^ chrs[24] == 0x004D)  # Line #41
s.add(0x0555 == (chrs[22] ^ 0x0539))

s.add(chrs[10] == chrs[7])

s.add(chrs[23] + chrs[8] == 0x00EB)
s.add(chrs[16] == chrs[17] + 0x0013)
s.add(chrs[19] == 0x006B)

s.add(chrs[20] + 0x01F5 == chrs[1] * 0x0005)

s.add(chrs[21] == chrs[22])

# n00bz
s.add(chrs[0] == ord(' '))
s.add(chrs[1] == ord('n'))
s.add(chrs[2] == ord('0'))
s.add(chrs[3] == ord('0'))
s.add(chrs[4] == ord('b'))
s.add(chrs[5] == ord('z'))
s.add(chrs[6] == ord('{'))
s.add(chrs[24] == ord('}'))

while s.check() == sat:
    m = s.model()
    print(''.join([chr(m[c].as_long()) if m[c] != None else '?' for c in chrs]))
    s.add(Or([chrs[i] != m[chrs[i]] for i in range(25)]))
'''
n00bz{3xc3l`y05}jsk1lls}
n00bz{3xc3lay0U|isk1lls}
n00bz{3xc3lay05|isk1lls}
n00bz{3xc3lay0u|isk1lls}
n00bz{3xc3l`y0U}jsk1lls}
n00bz{3xc3l`y0u}jsk1lls}
n00bz{3xc3lcy05~ksk1lls}
n00bz{3xc3lcy0u~ksk1lls}
n00bz{3xc3lcy0U~ksk1lls}
n00bz{3xc3l_y05r_sk1lls}
n00bz{3xc3l_y0Ur_sk1lls}
n00bz{3xc3l_y0ur_sk1lls}
'''
```

Take a guess, and it is correct.

```flag
n00bz{3xc3l_y0ur_sk1lls}
```
