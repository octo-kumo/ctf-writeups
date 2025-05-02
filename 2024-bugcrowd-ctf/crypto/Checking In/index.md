---
ai_date: '2025-04-27 05:13:40'
ai_summary: Periodic table cipher with element symbols replaced by their atomic numbers
ai_tags:
- cryptography
- substitution
- periodic-table
created: 2024-08-08T08:44
points: 125
updated: 2024-08-17T20:13
---

Periodic table cipher.

```python
from periodictable import elements

c = b'CMgHN{KClScHKOFSiN_AlPMgBLiScMgBK_MgFNaB_H_HePKK}'
for el in sorted(list(elements)[1:], key=lambda x: len(x.symbol), reverse=True):
    c = c.replace(el.symbol.encode(), bytes([el.number]))

print(bytes([i if i > 0x30 else ord('a')+i-1 for i in c]))
```

```flag
flag{squashing_molecules_like_a_boss}
```