---
ai_date: 2025-04-27 05:18:08
ai_summary: Reversing challenge involving iterative character modification based on ASCII range and bitwise operations
ai_tags:
  - rev
  - xor
  - bit-manipulation
created: 2024-07-20T23:23
points: 100
solves: 86
updated: 2025-07-14T09:46
---

After gaining access to almighty IDA, reversing had never been easier.
## decompile

```cpp [main()]
v3 = iterate(0);
v4 = iterate(v3);
v5 = iterate(v4);
v6 = iterate(v5);
v7 = iterate(v6);
v8 = iterate(v7);
v9 = iterate(v8);
v10 = iterate(v9);
v11 = iterate(v10);
v12 = iterate(v11);
...
```

```cpp [iterate()]
__int64 __fastcall iterate(int i)
{
  bool v1; // al
  unsigned __int8 chr; // [rsp+19h] [rbp-7h]
  bool v4; // [rsp+1Eh] [rbp-2h]

  chr = flag[i];
  v4 = (i & 1) != 0;
  v1 = chr > 0x60u && chr <= 0x7Au;
  flag[i] = ((((int)chr >> table2[iterate(int)::counter2]) | (chr << (8 - table2[iterate(int)::counter2]))) * v1
           + !v1 * (((chr << 6) | (chr >> 2)) ^ table1[iterate(int)::counter1]))
          * ((i & 1) == 0)
          + ((chr ^ table1[iterate(int)::counter1]) * v1 + !v1 * ((4 * chr) | (chr >> 6))) * ((i & 1) != 0);
  iterate(int)::counter1 = (v4 + iterate(int)::counter1) % 6;
  iterate(int)::counter2 = (v4 + iterate(int)::counter2) % 6;
  printf("%02x,", (unsigned __int8)flag[i]);
  return (unsigned int)(i + 1);
}
```

It appears that every iterate is just returning the next index, so it is just a for loop.
## rev
I first translated it into python with chatgpt.

```python
def try_flag(flag):
    flag = [ord(i) for i in flag]
    counter1 = 0
    counter2 = 0

    final_str = []

    def iterate(i):
        nonlocal counter1, counter2, final_str

        char = flag[i]
        v4 = (i & 1) != 0
        v1 = 0x60 < char <= 0x7A

        if v1:
            rotated = (char >> table2[counter2]) | (char << (8 - table2[counter2]))
            flag[i] = rotated & 0xFF  # Ensure byte size
        else:
            rotated = ((char << 6) | (char >> 2)) ^ table1[counter1]
            flag[i] = rotated & 0xFF  # Ensure byte size

        if (i & 1) == 0:
            flag[i] = flag[i] * int(v1)
        else:
            if v1:
                flag[i] = (char ^ table1[counter1]) & 0xFF  # Ensure byte size
            else:
                flag[i] = ((4 * char) | (char >> 6)) & 0xFF  # Ensure byte size

        counter1 = (v4 + counter1) % 6
        counter2 = (v4 + counter2) % 6
        # print(f"{flag[i]:02x},", end='')
        return i + 1

    i = 0
    while i < len(flag):
        i = iterate(i)
    print(' '.join([f"{i:02x}" for i in flag]))
    return flag
```

And found out that all characters are modified independently of each other, that's great.

```python
try_flag("nothing_here_lmao")
try_flag("oothing_here_lmao")
# 37 3d 8e 0c 96 1f d9 7d a1 31 93 13 00 3e ad 05 f6
# b7 3d 8e 0c 96 1f d9 7d a1 31 93 13 00 3e ad 05 f6
```

## attempt 1

```python
flag = [0 for i in target]
all_chars = [[] for i in target]
for i in tqdm(range(len(target))):
    for c in string.digits + string.ascii_letters + string.punctuation:
        flag[i] = ord(c)
        res = try_flag(flag)
        if res[i] == target[i]:
            all_chars[i].append(c)
    if len(all_chars[i]) == 0:
        if i == 4:
            flag[i] = ord('{')
        continue
    flag[i] = ord(all_chars[i][0])
    print(''.join(chr(c) for c in flag))
m = max([len(i) for i in all_chars])
for r in range(m):
    for i in range(len(all_chars)):
        if r < len(all_chars[i]):
            print(all_chars[i][r], end='')
        else:
            print('█', end='')
    print()
```

And... I am missing a bunch of characters, hrm.

```
ictf█mur█_than█1jway█_t0█c█n█rul█
█L███████████████████████L█████O█
```

## fix

Turns out chatgpt was being dumb, I had to manually fix the logic errors.
*sigh*

```python
def try_flag(flag):
    flag = [i for i in flag]

    counter1 = 0
    counter2 = 0

    final_str = []

    def iterate(i):
        nonlocal counter1, counter2, final_str
        char = flag[i]
        v4 = (i & 1) != 0
        v1 = 0x60 < char <= 0x7A
        if (i & 1) == 0:
            if v1:
                rotated = (char >> table2[counter2]) | (char << (8 - table2[counter2]))
                flag[i] = rotated & 0xFF  # Ensure byte size
            else:
                rotated = ((char << 6) | (char >> 2)) ^ table1[counter1]
                flag[i] = rotated & 0xFF  # Ensure byte size
        else:
            if v1:
                flag[i] = (char ^ table1[counter1]) & 0xFF  # Ensure byte size
            else:
                flag[i] = ((4 * char) | (char >> 6)) & 0xFF  # Ensure byte size
        counter1 = (v4 + counter1) % 6
        counter2 = (v4 + counter2) % 6
        # print(f"{flag[i]:02x},", end='')
        return i + 1

    i = 0
    while i < len(flag):
        i = iterate(i)
    return flag
```

```flag
ictf{m0r3_than_1jway5_t0_c0n7r0l}
█L████u███W█████_█$███W██L████uO█
```

yay!

After some guess and check, the flag is:

```flag
ictf{m0r3_than_1_way5_t0_c0n7r0l}
```

## solve script

```python
import string
from tqdm import tqdm
table1 = [0x52, 0x64, 0x71, 0x51, 0x54, 0x76]
table2 = [1, 3, 4, 2, 6, 5]

# Initialize counters
target = [0xb4, 0x31, 0x8e, 0x02, 0xaf, 0x1c, 0x5d, 0x23, 0x98, 0x7d, 0xa3, 0x1e, 0xb0, 0x3c, 0xb3, 0xc4,
          0xa6, 0x06, 0x58, 0x28, 0x19, 0x7d, 0xa3, 0xc0, 0x85, 0x31, 0x68, 0x0a, 0xbc, 0x03, 0x5d, 0x3d, 0x0b]
print(target)


def try_flag(flag):
    flag = [i for i in flag]

    counter1 = 0
    counter2 = 0

    final_str = []

    def iterate(i):
        nonlocal counter1, counter2, final_str
        char = flag[i]
        v4 = (i & 1) != 0
        v1 = 0x60 < char <= 0x7A
        if (i & 1) == 0:
            if v1:
                rotated = (char >> table2[counter2]) | (char << (8 - table2[counter2]))
                flag[i] = rotated & 0xFF  # Ensure byte size
            else:
                rotated = ((char << 6) | (char >> 2)) ^ table1[counter1]
                flag[i] = rotated & 0xFF  # Ensure byte size
        else:
            if v1:
                flag[i] = (char ^ table1[counter1]) & 0xFF  # Ensure byte size
            else:
                flag[i] = ((4 * char) | (char >> 6)) & 0xFF  # Ensure byte size
        counter1 = (v4 + counter1) % 6
        counter2 = (v4 + counter2) % 6
        # print(f"{flag[i]:02x},", end='')
        return i + 1

    i = 0
    while i < len(flag):
        i = iterate(i)
    return flag


flag = [0 for i in target]
all_chars = [[] for i in target]
for i in tqdm(range(len(target))):
    for c in string.digits + string.ascii_letters + string.punctuation:
        flag[i] = ord(c)
        res = try_flag(flag)
        if res[i] == target[i]:
            all_chars[i].append(c)
    if len(all_chars[i]) == 0:
        if i == 4:
            flag[i] = ord('{')
        continue
    flag[i] = ord(all_chars[i][0])
    print(''.join(chr(c) for c in flag))
m = max([len(i) for i in all_chars])
for r in range(m):
    for i in range(len(all_chars)):
        if r < len(all_chars[i]):
            print(all_chars[i][r], end='')
        else:
            print('█', end='')
    print()
```