---
ai_date: 2025-04-27 05:23:45
ai_summary: Obfuscated encryption using ASCII shifting and XOR, involving base conversion and modulo arithmetic
ai_tags:
  - xor
  - ascii-shifting
  - base-conversion
created: 2024-06-22T09:03
points: 128
solves: 402
updated: 2025-07-14T09:46
---

- [gio54321/lambdifier: Python obfuscation for the average lambda enjoyer (github.com)](https://github.com/gio54321/lambdifier)

```python
key = '16_10_13_x_6t_4_1o_9_1j_7_9_1j_1o_3_6_c_1o_6r'
key = (chr(int(c, 36) + 10) for c in key.split('_'))
dec = ''.join(chr((123 ^ ord(c))-9) for c in key)
print(dec)
```