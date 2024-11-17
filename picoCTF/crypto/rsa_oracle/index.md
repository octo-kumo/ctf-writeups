---
created: 2024-11-16T19:35
updated: 2024-11-16T19:41
---

$$
(x^e\mod{m})\times(2^e\mod{m})\equiv (2x)^e\mod{m}
$$

Only the flag itself is forbidden, so we can just decrease $2\times\text{flag}$ then divide it by 2 afterwards.

I went on a tangent and found the modulo too.

$$
m=\gcd(E(2)^{2}-E(4),E(3)^{2}-E(9))
$$

```python
from pwn import *
from Crypto.Util.number import long_to_bytes, bytes_to_long

enc = 873224563026311790736191809393138825971072101706285228102516279725246082824238887755080848591049817640245481028953722926586046994669540835757705139131212
conn = remote('titan.picoctf.net', 51523)


def d(cipher):
    conn.sendlineafter(b'decrypt.', b'D')
    conn.sendlineafter(b': ', str(cipher).encode())
    try:
        conn.recvuntil(b'mod n): ')
        dec = conn.recvline().strip().decode()
        if len(dec) % 2 != 0:
            dec = '0' + dec
        dec = bytes.fromhex(dec)
        return bytes_to_long(dec)
    except EOFError:
        return None


def e(cipher):
    conn.sendlineafter(b'decrypt.', b'E')
    conn.sendlineafter(b': ', long_to_bytes(cipher))
    conn.recvuntil(b'mod n) ')
    dec = conn.recvline().strip().decode()
    return int(dec)


# m = math.gcd(e(2)**2-e(4), e(3)**2-e(9))
# m = 5507598452356422225755194020880876452588463543445995226287547479009566151786764261801368190219042978883834809435145954028371516656752643743433517325277971

print(long_to_bytes(d((enc*e(2)))//2))
conn.close()

# 92d53
```

```bash
$ openssl enc -aes-256-cbc -d -in secret.enc
enter AES-256-CBC decryption password:
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
picoCTF{su((3ss_(r@ck1ng_r3@_92d53250}
```

```flag
picoCTF{su((3ss_(r@ck1ng_r3@_92d53250}
```
