---
created: 2024-06-28T22:58
updated: 2024-06-30T20:06
---

# Determined

The problem involves the determinant of a 5 by 5 matrix, where we can decide certain the value of cells, and some other values unknown.

Since I am not that big of a math person, I will solve it with `sympy`.

Furthermore, since we are allowed to change 9 cells, I will prompt the remote 9 times, each with 1 cell incremented, i.e.

```
2 1 1 1 1 1 1 1 1 
1 2 1 1 1 1 1 1 1 
1 1 2 1 1 1 1 1 1 
...
```

Record each output, and add it to my list of equations, after which solve for $p$ $q$ and $r$.

After solving them, we will simply apply RSA decryption.

```python
import sympy as sp
from pwn import *
from Crypto.Util.number import long_to_bytes
from itertools import permutations
p, q, r = sp.symbols('p q r', integer=True)
n = 158794636700752922781275926476194117856757725604680390949164778150869764326023702391967976086363365534718230514141547968577753309521188288428236024251993839560087229636799779157903650823700424848036276986652311165197569877428810358366358203174595667453056843209344115949077094799081260298678936223331932826351
c = 72186625991702159773441286864850566837138114624570350089877959520356759693054091827950124758916323653021925443200239303328819702117245200182521971965172749321771266746783797202515535351816124885833031875091162736190721470393029924557370228547165074694258453101355875242872797209141366404264775972151904835111

context.log_level = 'error'


def rem(nums):
    conn = remote('determined.chal.uiuc.tf', 1337, ssl=True)
    conn.recvuntil(b"M[0][2] = ")
    for i in nums:
        conn.sendline(str(i))
    conn.recvuntil(b"Have fun: ")
    num = int(conn.recvline().strip())
    conn.close()
    print(nums, "=>", num)
    return num


def inputs(nums):
    return [
        [p, 0, nums[0], 0, nums[1]],
        [0, nums[2], 0, nums[3], 0],
        [nums[4], 0, nums[5], 0, nums[6]],
        [0, q, 0, r, 0],
        [nums[7], 0, nums[8], 0, 0],
    ]


equations = []
for i in range(9):
    nums = [2 if i == j else 1 for j in range(9)]
    equations.append(sp.Eq(sp.Matrix(inputs(nums)).det(), rem(nums)))
solutions = sp.solve(equations, (p, q, r))
print("Solutions:", solutions)
p, q, r = solutions[0]
if int(p*q) != n:
    print("p*q!=n")
    exit()
n = int(p*q)
phi_n = int((p - 1) * (q - 1))

e = 65535
d = pow(e, -1, phi_n)
m = pow(c, d, n)
recovered_plain = long_to_bytes(m)
print(recovered_plain)
# uiuctf{h4rd_w0rk_&&_d3t3rm1n4t10n}
```
