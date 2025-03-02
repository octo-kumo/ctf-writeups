---
created: 2025-03-01T21:48
updated: 2025-03-01T22:03
---

There is a 10s delay between successive guesses, hrmm.

Well we can just brute force the seed, since `microsecond` can only be from `0-999999`.

```python
from pwn import *
r = remote('localhost', 55998)


def find_seed(history):
    candidates = []
    for seed in range(0, 999999):
        random.seed(seed)
        if all(random.randint(0, 54321) == i for i in history):
            candidates.append(seed)
    return candidates


r.sendlineafter(b'Enter your guess: ', b'0')
r.recvuntil(b'The number was: ')
numA = int(r.recvline().strip())
r.sendlineafter(b'Enter your guess: ', b'0')
r.recvuntil(b'The number was: ')
numB = int(r.recvline().strip())

seeds = find_seed([numA, numB])
if len(seeds) != 1:
    print('Seed not found')
    exit()

seed = seeds[0]
random.seed(seed)
a = random.randint(0, 54321)
b = random.randint(0, 54321)
c = random.randint(0, 54321)


r.sendlineafter(b'Enter your guess: ', str(c).encode())
r.interactive()
```

```flag
ATHACKCTF{littl3_5quid_with_a_littl3_533d}
```
