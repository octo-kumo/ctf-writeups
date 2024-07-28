---
created: 2024-07-27T20:56
updated: 2024-07-28T04:16
title: Not an active field for a reason
tags:
  - tree-parity-machine
  - neuro-crypto
points: 470
solves: 4
---

## Neural networks
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1722130753/2024/07/fa7fc5d404cff6a9e1b843b66b0c6740.png)

Tree parity machine are simple networks that takes $N\times K$ inputs and outputs a single binary output.

Two TPMs sync with each other in sort of a relationship.

Each time that they "act in sync" with each other they "learn" and over many iterations their weights will slowly equalize.

Basically they both learn how to act like the other.

So, how long does it take for them to get along?

## The relationship

```python
for _ in range(1000):
    x = np.random.randint(-25, 26, Alice.n * Alice.k)
    t1 = Alice.forward(x)
    t2 = Bob.forward(x)
    inputs.append(list(x))
    alice_taus.append(t1)
    bob_taus.append(t2)
    if t1 == t2:
        Alice.backward(Bob.tau)
        Bob.backward(Alice.tau)
    else:
        print(_, t1, t2, "mismatch")
```

By matching many pairs of random machine together we can determine that it takes roughly ~100 iterations for them to act in sync with each other, afterwards their weights might continue to change but would be in sync with each other, i.e. the relationship has been established.

### The third person

Now, we, the evil overlords, will introduce a third TCM, **C** to the relationship, C will try to learn from interactions of **A&B** and imitate their behaviour.

To do that, we will force C to encounter every event in the past that occurred to A&B, and learn from every encounter where A&B made the same choice, i.e. every interaction after the initial "getting to know each other" period.

This way, C will be learn from the lovely honeymoon A&B are having, and would be ignorant to all the hardships they had before that.

After 1 iteration however, C might not have fully equalized with A&B. That's ok, they are machines after all, we can just make C learn it again, and again (btw, we are actually **overfitting** over here).

After some testing it shows that 3 iterations over the entire history of the relationship is enough for C to become one with the pair.

C will relive the happy memories of A&B several times.

## Solve Script

```python
import hashlib
from machine import TreeParityMachine
import numpy as np
from Crypto.Cipher import AES
from pathlib import Path
ct = ""
alice_taus = []
bob_taus = []
inputs = []
path = Path(__file__).parent / "output.txt"
with open(path, "r") as f:
    exec(''.join(f.readlines()))

k, l, n = 7, 10, 10
Cim = TreeParityMachine(k, n, l, "hebian")
print("len(alice_taus)", len(alice_taus))

for I in range(10):
    for _ in range(100, 1000):
        t = Cim.forward(np.array(inputs[_]))
        if alice_taus[_] == bob_taus[_]:
            Cim.backward(np.array(alice_taus[_]))

sha256 = hashlib.sha256()
sha256.update(Cim.W.tobytes())
key = sha256.digest()
cipher = AES.new(key, AES.MODE_ECB)
print(cipher.decrypt(bytes.fromhex(ct)))
```

```flag
DEAD{Hamoor_added_AI_so_crypto_people_think_its_hard}
```
