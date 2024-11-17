---
created: 2024-11-16T15:26
updated: 2024-11-16T19:34
---

Just reverse the two operations.

```python
cipher = ''.join([chr(c//shared_key//311) for c in cipher])
print(dynamic_xor_encrypt(cipher[::-1], text_key)[::-1])
```

```flag
picoCTF{custom_d2cr0pt6d_8b41f976}
```
