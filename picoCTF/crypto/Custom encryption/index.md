---
ai_date: 2025-04-27 05:28:50
ai_summary: "Reversed operations: Custom XOR decryption followed by custom encryption"
ai_tags:
  - xss
  - custom-cipher
  - reverse-operations
created: 2024-11-16T15:26
updated: 2025-07-14T09:46
---

Just reverse the two operations.

```python
cipher = ''.join([chr(c//shared_key//311) for c in cipher])
print(dynamic_xor_encrypt(cipher[::-1], text_key)[::-1])
```

```flag
picoCTF{custom_d2cr0pt6d_8b41f976}
```