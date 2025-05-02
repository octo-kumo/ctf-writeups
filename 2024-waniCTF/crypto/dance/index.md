---
ai_date: '2025-04-27 05:23:11'
ai_summary: Brute-forced token generation and encryption to find the flag
ai_tags:
- brute
- hash
- sha256
created: 2024-06-23T05:36
points: 205
solves: 85
updated: 2024-08-05T09:48
---

## Tokens
A realization was made that, the tokens only have `60*60*11` possible states.
I can brute force it easily.

## Encryption / Decryption
Another realization was made when playing around that, the encryption function is also the decryption function.

```python
for i in range(10):
    ciphertext = cipher.encrypt(ciphertext)
    print('ciphertext:', ciphertext.hex())
```

```
ciphertext: e642d9b8afdbe8ad184938e529bdc102
ciphertext: d06f51bfb2c5fa4373d68d85ae58009f
ciphertext: e642d9b8afdbe8ad184938e529bdc102
ciphertext: d06f51bfb2c5fa4373d68d85ae58009f
ciphertext: e642d9b8afdbe8ad184938e529bdc102
ciphertext: d06f51bfb2c5fa4373d68d85ae58009f
```

## Solve

```python
from tqdm import tqdm
import hashlib

from mycipher import MyCipher

text = bytes.fromhex('061ff06da6fbf8efcd2ca0c1d3b236aede3f5d4b6e8ea24179')
def make_token(data1: str, data2: str):
    sha256 = hashlib.sha256()
    sha256.update(data1.encode())
    right = sha256.hexdigest()[:20]
    sha256.update(data2.encode())
    left = sha256.hexdigest()[:12]
    token = left + right
    return token

for minutes in tqdm(range(60)):
    for sec in range(60):
        for r in range(11):
            username = "gureisya"
            data1 = f'user: {username}, {minutes}:{sec}'
            data2 = f'{username}'+str(r)
            token = make_token(data1, data2)

            sha256 = hashlib.sha256()
            sha256.update(token.encode())
            key = sha256.hexdigest()[:32]
            nonce = token[:12]
            cipher = MyCipher(key.encode(), nonce.encode())
            flag = cipher.encrypt(text)
            if flag.startswith(b'FLAG{'):
                print(flag)
                exit()
```

```
'FLAG{d4nc3_l0b0t_d4nc3!!}'
```