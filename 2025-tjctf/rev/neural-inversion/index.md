---
created: 2025-06-06T20:08
updated: 2025-06-08T12:02
solves: 90
points: 420
---

GPT solved it lol.

```python
import numpy as np

data = np.load("model.npz")
W1, b1 = data["W1"], data["b1"]
W2, b2 = data["W2"], data["b2"]
y = data["y"]

z2 = np.log(y / (1 - y))
a1 = np.linalg.inv(W2).dot(z2 - b2)
z1 = np.log(a1 / (1 - a1))
x_rec = np.linalg.inv(W1).dot(z1 - b1)
ints = np.round(x_rec * 127).astype(int)
flag = "".join(chr(i) for i in ints)
print(flag)
```

```flag
tjctf{m0d3l_1nv3rs10n_f9a93qw}
```
