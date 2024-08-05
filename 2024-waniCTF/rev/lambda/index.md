---
created: 2024-06-22T09:03
updated: 2024-08-05T09:51
solves: 402
points: 128
---

 - [gio54321/lambdifier: Python obfuscation for the average lambda enjoyer (github.com)](https://github.com/gio54321/lambdifier)

```python
key = '16_10_13_x_6t_4_1o_9_1j_7_9_1j_1o_3_6_c_1o_6r'
key = (chr(int(c, 36) + 10) for c in key.split('_'))
dec = ''.join(chr((123 ^ ord(c))-9) for c in key)
print(dec)
```
