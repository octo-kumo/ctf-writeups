---
created: 2024-12-14T01:15
updated: 2025-01-05T19:48
points: 825
---

It is simple crypto, the timestamp part simply alternates between mode 1 and mode 2, even if its random its still ok anyways.

Solution is just reverse operations.

```python
import base64
import random
import string
import itertools


p1 = bytes.fromhex("00071134013a3c1c00423f330704382d00420d331d04383d00420134044f383300062f34063a383e0006443310043839004315340314382f004240331c043815004358331b4f3830")
p2 = bytes.fromhex("5d1f486e4d49611a5d1e7e6e4067611f5d5b196e5b5961405d1f7a695b12614e5d58506e4212654b5d5b196e4067611d5d5b726e4649657c5d5872695f12654d5d5b4c6e4749611b")


def generate_key(seed, length=16):
    random.seed(seed)
    key = ''.join(random.choice(string.ascii_letters + string.digits) for _ in range(length))
    return key


def xor_cipher(text, key):
    return bytes([c ^ key for c in text])


def polyalphabetic_decrypt(ciphertext, key):
    ciphertext = base64.b64decode(ciphertext).decode()
    key_length = len(key)
    plaintext = []
    for i, char in enumerate(ciphertext):
        key_char = key[i % key_length]
        decrypted_char = chr((ord(char) - ord(key_char)) % 256)
        plaintext.append(decrypted_char)
    return ''.join(plaintext)


def decrypt(cipher, k1, k2):
    im = xor_cipher(cipher, k2)
    try:
        key = generate_key(k1)
        dec = polyalphabetic_decrypt(im, key)
        if all(c in string.printable for c in dec):
            return dec
    except:
        pass


def crack1(cipher):
    return [(i, 42) for i in range(1, 1001) if decrypt(cipher, i, 42)]


def crack2(cipher):
    return [(42, i) for i in range(1, 256) if decrypt(cipher, 42, i)]


keys11, keys12 = crack1(p1), crack2(p1)
keys21, keys22 = crack1(p2), crack2(p2)
key1 = keys11 if keys11 else keys12
key2 = keys22 if keys22 else keys21

for k1, k2 in itertools.product(key1, key2):
    print(decrypt(p1, k1[0], k1[1])+decrypt(p2, k2[0], k2[1]))
```

```flag
HTB{timestamp_based_encryption_is_so_secure_i_promise}
```
