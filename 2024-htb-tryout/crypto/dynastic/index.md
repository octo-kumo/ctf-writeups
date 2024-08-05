---
created: 2024-07-17T01:12
updated: 2024-08-04T19:32
solves: 462
points: 900
---

Single character change crypto.

```python
def to_identity_map(a):
    return ord(a) - 0x41


def from_identity_map(a):
    return chr(a % 26 + 0x41)


def decrypt(m):
    c = ''
    for i in range(len(m)):
        ch = m[i]
        if not ch.isalpha():
            ech = ch
        else:
            chi = to_identity_map(ch)
            # ech = from_identity_map(chi + i)
            ech = from_identity_map(chi - i)
        c += ech
    return c


C = "DJF_CTA_SWYH_NPDKK_MBZ_QPHTIGPMZY_KRZSQE?!_ZL_CN_PGLIMCU_YU_KJODME_RYGZXL"

print(f'HTB{{{decrypt(C)}}}')
```
