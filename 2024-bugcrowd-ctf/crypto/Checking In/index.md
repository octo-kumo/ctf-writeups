---
created: 2024-08-08T08:44
updated: 2024-08-08T09:49
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