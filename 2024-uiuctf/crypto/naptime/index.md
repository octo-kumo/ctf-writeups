---
ai_date: 2025-04-27 05:21:46
ai_summary: Flag encryption using bitwise addition, solved with Z3 by brute forcing binary combinations
ai_tags:
  - bitwise
  - brute-force
  - xor
created: 2024-06-28T22:58
points: 363
solves: 180
updated: 2025-07-14T09:46
---

After reading through the challenge, we can see that the encryption method simply adds numbers from `a[i]` where `i`th bit is `1`.
This means its sort of $e=b_0a_0+b_1a_1+\dots$.

```python
def enc(flag, a, n):
    bitstrings = []
    for c in flag:
        # c -> int -> 8-bit binary string
        bitstrings.append(bin(ord(c))[2:].zfill(8))
    ct = []
    for bits in bitstrings:
        curr = 0
        for i, b in enumerate(bits):
            if b == "1":
                curr += a[i]
        ct.append(curr)
```

We can hence solve it with Z3.

```python
from z3 import *
a = [66128, 61158, 36912, 65196, 15611, 45292, 84119, 65338]
ct = [273896, 179019, 273896, 247527, 208558, 227481, 328334, 179019, 336714, 292819, 102108, 208558, 336714, 312723, 158973, 208700, 208700, 163266, 244215,
      336714, 312723, 102108, 336714, 142107, 336714, 167446, 251565, 227481, 296857, 336714, 208558, 113681, 251565, 336714, 227481, 158973, 147400, 292819, 289507]
for num in ct:
    a1, a2, a3, a4, a5, a6, a7, a8 = Bools('a1 a2 a3 a4 a5 a6 a7 a8')
    curr = 0
    for i, b in enumerate([a1, a2, a3, a4, a5, a6, a7, a8]):
        curr += If(b, a[i], 0)
    s = Solver()
    s.add(curr == num)
    if s.check() == sat:
        values = [s.model()[a] for a in [a1, a2, a3, a4, a5, a6, a7, a8]]
        values = ''.join(['1' if values[i] else '0' for i in range(8)])
        ascii_char = chr(int(values, 2))
        print(ascii_char, end='')
    else:
        raise Exception("Unsat")
```

```flag
uiuctf{i_g0t_sleepy_s0_I_13f7_th3_fl4g}
```