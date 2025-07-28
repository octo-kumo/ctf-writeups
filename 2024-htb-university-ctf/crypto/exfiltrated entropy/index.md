---
ai_date: 2025-04-27 05:16:53
ai_summary: Writeup describes a vulnerability in a solver script that exploits a leaked LCG (Linear Congruential Generator) state, specifically the last byte, to determine the seed. The total number of seeds is limited to 256 due to the state size.
ai_tags:
  - lcg
  - state-leak
  - predictability
created: 2024-12-14T01:35
points: 900
updated: 2025-07-14T09:46
---

I was writing a solver that can solve the SEED based on a stream of leaked last byte, possibly gotten via guessing the key stream, but oh well.

THE STATE ITSELF IS LEAKED DIRECTLY LMAO.

## useless solver

```python [solver.py]
def rev(i, a, b, m, s):
    for _ in range(i):
        s -= b
        b = (b * a) % m
    s %= m
    s &= 0xff  # this is equal to state & 0xff
    return s


def validate(seed, amt=2048):
    m1 = LCG(seed)
    m2 = LCG()
    for _ in range(amt):
        if m1._next() & 0xff != m2._next() & 0xff:
            return _
    return True


lcg = LCG()
X = Int('X')
solver = Solver()
solver.add(And(0 <= X, X < m))
for i, s in enumerate(lcg.generate_key(1)):
    res = rev(i+1, a, b, m, s)
    solver.add((pow(a, i+1, m) * X % m) % 256 == res)
print(SEED, validate(SEED))
solutions = []
while solver.check() == sat:
    model = solver.model()
    solution = model[X].as_long()
    print("solution", solution, "valid?", validate(solution))
    solutions.append(solution)
    solver.add(X != solution)
```

It appears to be finding a lot of seed that give correct answers all the way up to 2048.

... And it appears that only 1 byte is needed, which means the state itself is only relevant to the last byte anyways.

Which means the total number of different seeds is only 256.

## smart solver

```python
import base64
import json
import pyshark
from params import *
from pwn import *
cap = pyshark.FileCapture('traffic18112024.pcap', display_filter="tcp")
inputs = []
outputs = []


class LCG:
    def __init__(self, seed):
        self.state = seed
        self.a = a
        self.b = b
        self.m = m

    def _next(self):
        self.state = (self.a * self.state + self.b) % self.m
        return self.state

    def generate_key(self, l):
        return bytes([self._next() & 0xff for _ in range(l)])

    def generate_packet_uuid(self):
        return hex(self._next())

    def encrypt(self, msg):
        key = self.generate_key(len(msg))
        return xor(msg, key)

    def decrypt(self, msg):
        return self.encrypt(msg)


lcg = LCG(1)

partial = None
cipher = b""
for pkt in cap:
    try:
        if pkt.tcp.payload:
            p = bytes.fromhex(pkt.tcp.payload.replace(':', '')).decode()
            if p[0] == '{':
                p = json.loads(p)
                print(lcg.decrypt(base64.b64decode(p['cmd'])))
                if 'id' in p:
                    lcg.state = int(p['id'], 16)
            else:
                lcg.decrypt(base64.b64decode(p))
    except AttributeError:
        pass
```

```
b'\xb12\x85O\x93\x13'
b'\xf7j'
b'cat /etc/passwd'
b'lsb_release -a'
b'echo "H4sIAAE4/WYAA+1YOXLDMAzs9Ro+My0bPzAvyYwnlsjF7gKkncJJOGoIgMDipKT2efsYn9YaUmAfn4LEfcG2EVsgNvOiRrMogvW1okCaPD5v/f6MQZ4pj00HxkxRLNR+QukjuiB10roA1Ikzyvcl8UxNz0WNyaPFeMyuIaldMVCnHkH9tg2pVmWPUl3uRsd0FzGqaTmtckUXFbf7dSeqRJdpZzNGOz2J3T/w5p6ZepJVZinvor1ZRM2IuFDw9GOXe53y2FP7smsbDF0belpmtcFPAoVlcKRre9WpQK6LZCOVMXX3mn6BlfIu2QQ8U1QNHFEBaaIpgeCPCXftvhO0V2adqHXNt2VLmNvKPcdzVUQ587ELkVIgxYeManV30WmAhMW95EVBYn4FWUmZtmKdkqPSv1U8jUox8lif7/BnHbZIKZDiM9/+nRxoVDUSFveSl1qqIiooyQT3YrzjTpFYiOXR/sL69/L3rNFLMpL5soKK6S6znL275rvPqhW26zCGWD5GQiGURlAxwy3Yg2RBZHexKZmHFO0DJTN6yL8EPdJpujEshVeMAIz5OhOmu0YweIAM6he5Eo7V/l1FfZD/QVs5oUe8R2c1YIwAoeV0yeEGxUlRz5kVredbCABFyotcWThKHLcmzK+3lRlCOzbkNyZbuy3RSkS7WSWuD4I4cfQQ4smw37GlI9nPi7yfzk8P2g0xl7LEfjSbLK8xgQyVazxXxnQkiQCi0oJ7DHzF/UzldLFEr2hOaS0T45HBz4nuyPj+1c/wEjhlXBsYdAy2gxPk2RiSAZSqRy1zy1feZ+duuB7NUH8MzAHk+Z8JhmcglLBs2Fb+LgaByDJaRaWaC7D1eX+nBb1XLW5Yb+Uz5Hrb6ffyWjTV2voCIBKnivUiAAA=" | base64 -d | gunzip'
b'exit'
```

### the bash command

```bash
echo "H4sIAAE4/WYAA+1YOXLDMAzs9Ro+My0bPzAvyYwnlsjF7gKkncJJOGoIgMDipKT2efsYn9YaUmAfn4LEfcG2EVsgNvOiRrMogvW1okCaPD5v/f6MQZ4pj00HxkxRLNR+QukjuiB10roA1Ikzyvcl8UxNz0WNyaPFeMyuIaldMVCnHkH9tg2pVmWPUl3uRsd0FzGqaTmtckUXFbf7dSeqRJdpZzNGOz2J3T/w5p6ZepJVZinvor1ZRM2IuFDw9GOXe53y2FP7smsbDF0belpmtcFPAoVlcKRre9WpQK6LZCOVMXX3mn6BlfIu2QQ8U1QNHFEBaaIpgeCPCXftvhO0V2adqHXNt2VLmNvKPcdzVUQ587ELkVIgxYeManV30WmAhMW95EVBYn4FWUmZtmKdkqPSv1U8jUox8lif7/BnHbZIKZDiM9/+nRxoVDUSFveSl1qqIiooyQT3YrzjTpFYiOXR/sL69/L3rNFLMpL5soKK6S6znL275rvPqhW26zCGWD5GQiGURlAxwy3Yg2RBZHexKZmHFO0DJTN6yL8EPdJpujEshVeMAIz5OhOmu0YweIAM6he5Eo7V/l1FfZD/QVs5oUe8R2c1YIwAoeV0yeEGxUlRz5kVredbCABFyotcWThKHLcmzK+3lRlCOzbkNyZbuy3RSkS7WSWuD4I4cfQQ4smw37GlI9nPi7yfzk8P2g0xl7LEfjSbLK8xgQyVazxXxnQkiQCi0oJ7DHzF/UzldLFEr2hOaS0T45HBz4nuyPj+1c/wEjhlXBsYdAy2gxPk2RiSAZSqRy1zy1feZ+duuB7NUH8MzAHk+Z8JhmcglLBs2Fb+LgaByDJaRaWaC7D1eX+nBb1XLW5Yb+Uz5Hrb6ffyWjTV2voCIBKnivUiAAA=" | base64 -d | gunzip
```

The script evaluates to ASCII art.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1734162220/2024/12/3592a4557b7105789760b49369804cd9.png)

```flag
HTB{still_not_convinced_about_LCG_security?}
```