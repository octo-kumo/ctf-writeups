---
created: 2024-07-20T01:21
updated: 2024-07-21T18:46
tags:
  - crc32
  - salsa20
solves: 101
points: 100
---

I made two important discoveries.
1. Salsa20 is a streaming cipher, which means it will produce a fixed key stream
2. crc32 will change in the same way, if the same bytes are changed
3. `json.dumps` produces a space behind each comma

## salsa20

```python
cipher = Salsa20.new(key=KEY)
c1 = cipher.encrypt(b"abcdefguser")
cipher = Salsa20.new(key=KEY, nonce=cipher.nonce)
c2 = cipher.encrypt(b"abcdefgroot")
print(xor(c1, c2))
print(xor(b"abcdefguser", b"abcdefgroot"))
print(Salsa20.new(key=KEY, nonce=cipher.nonce).decrypt(xor(c1, xor(b"abcdefguser", b"abcdefgroot"))))
# b'\x00\x00\x00\x00\x00\x00\x00\x07\x1c\n\x06'
# b'\x00\x00\x00\x00\x00\x00\x00\x07\x1c\n\x06'
# b'abcdefgroot'
```

Like all other streaming ciphers, salsa20 produces "cipher key" $E$ which will be used as $E_i \oplus P_i = C_i$ to produce ciphertext $C$.
We can hence modify the ciphertext knowing parts of the plaintext.

```python
dummy = json.dumps({'user': 'user', 'command': "flag", 'nonce': token_hex(8)}).encode('ascii')
index = dummy.index(b'user",')
ciphertext = xor_bytes_range(ciphertext, index, xor(b'user', b'root'))
```

## crc32

```python
nonce = token_hex(8)
dummy1 = b'{"user": "user", "command": "cmd", "nonce": "'+nonce.encode()+b'"}'
dummy2 = b'{"user": "root", "command": "flag","nonce": "'+nonce.encode()+b'"}'
print(nonce)
print(crc32(dummy1) ^ crc32(dummy2))

nonce = token_hex(8)
dummy1 = b'{"user": "user", "command": "cmd", "nonce": "'+nonce.encode()+b'"}'
dummy2 = b'{"user": "root", "command": "flag","nonce": "'+nonce.encode()+b'"}'
print(nonce)
print(crc32(dummy1) ^ crc32(dummy2))
# 2030236186
# 2030236186
```

As we can observe, even though different tokens were used, as long as the change in bytes were identical, $crc(a) \oplus crc(b) = C$.
We can hence forge a new crc with xor.

```python
dummy1 = b'{"user": "user", "command": "cmd", "nonce": "ebad1837df0b74e9"}'
dummy2 = b'{"user": "root", "command": "flag","nonce": "ebad1837df0b74e9"}'
modifier = crc32(dummy1) ^ crc32(dummy2)
checksum = checksum ^ modifier
```

## the space
As you may have noticed, in the crc section I took advantage of the extra space behind `"cmd", ` to fit a four character command.

## solve script
Putting them together we construct the solve script.

```python
from pwn import *
from secrets import token_bytes, token_hex
from zlib import crc32
import json
from Crypto.Cipher import Salsa20
from Crypto.Util.number import bytes_to_long, long_to_bytes

context.log_level = 'error'


def xor_bytes_range(data, start_index, xor_value):
    data_array = bytearray(data)
    for i in range(len(xor_value)):
        data_array[start_index + i] ^= xor_value[i]
    return bytes(data_array)


def modify_plaintext(packet):
    packet = bytes.fromhex(packet)
    nonce = packet[:8]
    checksum = bytes_to_long(packet[8:12])
    ciphertext = packet[12:]

    dummy1 = b'{"user": "user", "command": "cmd", "nonce": "ebad1837df0b74e9"}'
    dummy2 = b'{"user": "root", "command": "flag","nonce": "ebad1837df0b74e9"}'
    modifier = crc32(dummy1) ^ crc32(dummy2)
    ciphertext = xor_bytes_range(ciphertext, dummy1.index(b'user",'), xor(b'user', b'root'))
    ciphertext = xor_bytes_range(ciphertext, dummy1.index(b'cmd", '), xor(b'cmd", ', b'flag",'))

    packet = nonce + long_to_bytes(checksum ^ modifier, 4) + ciphertext
    return packet.hex()


conn = remote('tango.chal.imaginaryctf.org', 1337)
conn.sendline(b"E")
conn.sendline(b"cmd")
conn.recvuntil(b'Your encrypted packet is: ')
token = conn.recvuntil(b"\n", drop=True)
token = modify_plaintext(token.decode()).encode()
conn.sendline(b"R")
conn.sendline(token)
conn.interactive()

# [E]ncrypt a command
# [R]un a command
# [Q]uit
# > Your encrypted packet (hex): ictf{F0xtr0t_L1m4_4lph4_G0lf}
# [E]ncrypt a command
# [R]un a command
# [Q]uit
# >
```

```flag
ictf{F0xtr0t_L1m4_4lph4_G0lf}
```
