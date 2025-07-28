---
ai_date: 2025-04-27 05:23:05
ai_summary: Brute-forced AES encryption by iterating through key and IV values to find the correct combination that decrypted the flag.
ai_tags:
  - brute
  - aes
  - cipher
created: 2024-06-22T01:50
points: 125
solves: 453
tags:
  - aes
updated: 2025-07-14T09:46
---

I don't know AES, but I know brute force.

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import hashlib

enc = b'\x16\x97,\xa7\xfb_\xf3\x15.\x87jKRaF&"\xb6\xc4x\xf4.K\xd77j\xe5MLI_y\xd96\xf1$\xc5\xa3\x03\x990Q^\xc0\x17M2\x18'
flag_hash = "6a96111d69e015a07e96dcd141d31e7fc81c4420dbbef75aef5201809093210e"
base_key = b'the_enc_key_is_'
base_iv = b'my_great_iv_is_'

for i in range(256):
    for j in range(256):
        key = base_key + bytes([i])
        iv = base_iv + bytes([j])
        try:
            cipher = AES.new(key, AES.MODE_CBC, iv)
            decrypted_msg = unpad(cipher.decrypt(enc), 16)
            decrypted_hash = hashlib.sha256(decrypted_msg).hexdigest()
            if decrypted_hash == flag_hash:
                print(f'Found matching key and iv: key = {key}, iv = {iv}')
                print(f'Decrypted message: {decrypted_msg.decode()}')
                break
        except (ValueError, KeyError):
            continue

# Found matching key and iv: key = b'the_enc_key_is_$', iv = b'my_great_iv_is_O'
# Decrypted message: FLAG{7h3_f1r57_5t3p_t0_Crypt0!!}
```