---
ai_date: 2025-08-17 20:46:37
ai_summary: Exploited pickle protocol differences between Python and C implementations
  to bypass restrictions
ai_tags:
- pickle
- protocol
- exploit
created: 2025-08-16T08:40
points: 100
solves: 105
title: Discrepancy
updated: 2025-08-17T20:46
---

We are only allowed 8 bytes of input that has to somehow give different results in `pickle` `_pickle` and `pickletools`.

I'm not gonna do this manually so let's brute force it.

https://github.com/python/cpython/blob/3.13/Lib/pickletools.py#L1153

Just brute forced length 2 ones, then cherry picked op codes that worked and looked nice and did a longer brute force with a subset of opcodes.

```python
import io, itertools, time
from io import BytesIO
import pickle, _pickle, pickletools
from contextlib import redirect_stdout

def py_ok(data: bytes) -> bool:
    class SafePyUnpickler(pickle._Unpickler):
        def find_class(self, module_name, global_name):
            raise RuntimeError("blocked")
    try:
        SafePyUnpickler(BytesIO(data)).load()
        return True
    except Exception:
        return False

def c_ok(data: bytes) -> bool:
    class SafeCUnpickler(_pickle.Unpickler):
        def find_class(self, module_name, global_name):
            raise RuntimeError("blocked")
    try:
        SafeCUnpickler(BytesIO(data)).load()
        return True
    except Exception:
        return False

def dis_ok(data: bytes) -> bool:
    try:
        f = io.StringIO()
        with redirect_stdout(f):
            pickletools.dis(data)
        return True
    except Exception:
        return False
answers = dict()
alphabet = [0x88, 0x2e, 0x28, 0x90, 0x8f, 0x62, 0x61, 0x80, 0x95, 0x00, 0xff, 0x01, 0x7f]

def test_bytes(b):
    s = bytes(b)[:8]
    py = py_ok(s); c = c_ok(s); d = dis_ok(s)
    return (py,c,d)

start = time.time()
checked = 0
for L in range(1, 9):
    print(f"Length {L}...")
    for tup in itertools.product(alphabet, repeat=L):
        checked += 1
        candidate = bytes(tup)[:8]
        res = test_bytes(candidate)
        if answers.get(res) is None:
            answers[res] = candidate
            print(f"Found {res} -> {candidate.hex()}")
            if len(answers) == 2**3:
                print("Found all answers!")
                exit(0)
```

```
Length 1...
Found (False, False, False) -> 88
Length 2...
Found (True, True, True) -> 882e
Found (False, False, True) -> 282e
Length 3...
Found (True, True, False) -> 88882e
Length 4...
Found (False, True, True) -> 8828902e
Found (True, False, True) -> 888f622e
Length 5...
Found (False, True, False) -> 888828902e
Found (True, False, False) -> 88888f622e
Found all answers!
```

```python
# ncat --ssl discrepancy.chals.sekai.team 1337
from pwn import *

answers = {(False, False, False): b'\x88', (True, True, True): b'\x88.', (False, False, True): b'(.', (True, True, False): b'\x88\x88.', (False, True, True): b'\x88(\x90.', (True, False, True): b'\x88\x8fb.', (False, True, False): b'\x88\x88(\x90.', (True, False, False): b'\x88\x88\x8fb.'}
conn = remote("discrepancy.chals.sekai.team", 1337, ssl=True)
req = [(True,True,False),
       (False,True,True),
       (True,False,True),
       (False,False,True),
       (False,True,False)]
for res in req:
    print(conn.recvline())
    conn.sendline(answers[res].hex().encode())

print(conn.recvall().decode())

# SEKAI{p1ckleeeeeeeee_3a01fea10fb01a88c1cd554e7372f21ced43b497}
```

```flag
SEKAI{p1ckleeeeeeeee_3a01fea10fb01a88c1cd554e7372f21ced43b497}
```