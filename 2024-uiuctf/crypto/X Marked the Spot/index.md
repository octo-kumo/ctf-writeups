---
ai_date: '2025-04-27 05:21:56'
ai_summary: Flag encrypted with XOR and cycling key; used known flag portion to derive
  key, then XOR decryption
ai_tags:
- xor
- cycling-key
- known-plaintext
created: 2024-06-28T22:58
points: 93
solves: 531
updated: 2024-11-23T20:30
---

We observe that the flag is encrypted with XOR with a cycling key of length 8.
And since we already know 8 characters of the flag (which happens to be length 48), we can determine the key by `cipher ^ plain = key`.

```
    7 chars     ...
key 01234567012 ... 567
txt uiuctf{???? ... ??}
```

```python
hints = b'uiuctf{}'

with open("ct", "rb") as ct_file:
    ct = list(ct_file.read())

res = []
for i in range(8):
    extracted = ct[i::8]
    k = hints[i] ^ (extracted[0] if i < 7 else extracted[-1])
    res += [x ^ k for x in extracted]
print(b"".join(bytes(res[i::6]) for i in range(6)))
```