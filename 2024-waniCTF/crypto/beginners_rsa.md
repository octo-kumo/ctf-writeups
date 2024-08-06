---
created: 2024-06-22T01:47
updated: 2024-08-05T19:29
points: 121
solves: 530
tags:
  - rsa
---

The big number can be factorized with online tools [Integer factorization calculator (alpertron.com.ar)](https://www.alpertron.com.ar/ECM.HTM)

```python
from Crypto.Util.number import *

n = 317903423385943473062528814030345176720578295695512495346444822768171649361480819163749494400347
e = 65537
enc = 127075137729897107295787718796341877071536678034322988535029776806418266591167534816788125330265

p=9953162929836910171
q=11771834931016130837
r=12109985960354612149
s=13079524394617385153
a=17129880600534041513

phi_n = (p - 1) * (q - 1) * (r - 1) * (s - 1) * (a - 1)
d = inverse(e, phi_n)
m = pow(enc, d, n)

decrypted_flag = long_to_bytes(m)
print(f'Decrypted flag: {decrypted_flag.decode()}')
# Decrypted flag: FLAG{S0_3a5y_1254!!}
```
