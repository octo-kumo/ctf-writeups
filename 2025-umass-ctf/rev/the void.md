---
ai_date: '2025-04-27 05:27:51'
ai_summary: XOR-based encryption with a fixed XOR key
ai_tags:
- xor
- xor-key
- brute-force
created: 2025-04-18T18:15
points: 486
solves: 20
title: the void
updated: 2025-04-20T20:27
---

It looks like the program is just doing random things with our input then calling `memcmp` with some answer in memory...

## analysis
Adding a breakpoint at `memcmp` we can analyze what is actually being compared to the answer, ie what has our input become.

```
ZZZAyxwvutsrqponmlkjihgfedcbazyxwvutsrqponmlkjihgfedcba
122 122 122 97 121 120 119 118 117 116 115 114 113 112 111 110 109 108 107 106 105 104 103 102 101 100 99 98 97 122 121 ...

result
98 99 100 32 119 119 119 119 119 119 103 103 103 103 103 103 113 113 113 113 113 113 113 113 113 113 97 97 97 39  ...
```

```python
# plain text
aaa = "a"*55
zzz = "z"*55
ZZZ = "Z"*55
BAA = "BBBBBBBBBBBBBBBBBBBBBBAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"
ABB = "AAAAAAAAAAAAAAAAAAAAAABBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB"
rand = "qwoidnoqwidbefiqnpwindiqniqwnidnqwiddqwjovbq12i0-anwicn"
# cipher text
aaa_result = [73, 80, 81, 82, 83, 84, 85, 86, 87, 88, 73, 80, 81, 82, 83, 84, 89, 96, 97, 98, 99, 100, 101, 102, 103, 104, 89, 96, 97, 98, 99, 100, 105, 112, 113, 114, 115, 116, 117, 118, 119, 120, 105, 112, 113, 114, 115, 116, 121, 128, 129, 130, 131, 132, 133]
zzz_result = [116, 117, 118, 119, 120, 121, 128, 129, 130, 131, 116, 117, 118, 119, 120, 121, 132, 133, 134, 135, 136, 137, 144, 145, 146, 147, 132, 133, 134, 135, 136, 137, 148, 149, 150, 151, 152, 153, 0, 1, 2, 3, 148, 149, 150, 151, 152, 153, 4, 5, 6, 7, 8, 9, 16]
ZZZ_result = [66, 67, 68, 69, 70, 71, 72, 73, 80, 81, 66, 67, 68, 69, 70, 71, 82, 83, 84, 85, 86, 87, 88, 89, 96, 97, 82, 83, 84, 85, 86, 87, 98, 99, 100, 101, 102, 103, 104, 105, 112, 113, 98, 99, 100, 101, 102, 103, 114, 115, 116, 117, 118, 119, 120]
BAA_result = [24, 25, 32, 33, 34, 35, 36, 37, 38, 39, 24, 25, 32, 33, 34, 35, 40, 41, 48, 49, 50, 51, 51, 52, 53, 54, 39, 40, 41, 48, 49, 50, 55, 56, 57, 64, 65, 66, 67, 68, 69, 70, 55, 56, 57, 64, 65, 66, 71, 72, 73, 80, 81, 82, 83]
ABB_result = [23, 24, 25, 32, 33, 34, 35, 36, 37, 38, 23, 24, 25, 32, 33, 34, 39, 40, 41, 48, 49, 50, 52, 53, 54, 55, 40, 41, 48, 49, 50, 51, 56, 57, 64, 65, 66, 67, 68, 69, 70, 71, 56, 57, 64, 65, 66, 67, 72, 73, 80, 81, 82, 83, 84]

rand_result = [101, 114, 101, 96, 86, 103, 105, 114, 121, 102, 82, 81, 85, 87, 97, 112, 114, 117, 131, 112, 118, 103, 115, 130, 128, 118, 117, 130, 116, 112, 102, 119, 133, 146, 121, 117, 118, 144, 151, 133, 145, 153, 112, 134, 35, 37, 129, 37, 5, 128, 148, 4, 145, 134, 152]
ans = [0, 33, 34, 4, 119, 82, 48, 17, 112, 41, 71, 1, 102, 113, 97, 52, 103, 19, 101, 51, 97, 85, 36, 35, 36, 35, 87, 117, 120, 19, 105, 129, 36, 80, 105, 115, 151, 114, 135, 48, 117, 64, 52, 104, 87, 134, 151, 41, 136, 84, 100, 87, 0, 56, 145]
ans = [a^0x61 for a in ans]
print(bytes(ans))
```

I did find some patterns but they were not helpful.

```python
for i in range(55):
  print(ord(aaa[i])-aaa_result[i]+i, end=",")
print()
for i in range(55):
  print(ord(zzz[i])-zzz_result[i]+i, end=",")
print()
for i in range(55):
  print(ord(ZZZ[i])-ZZZ_result[i]+i, end=",")
print()
for i in range(55):
  print(ord(rand[i])-rand_result[i]+i, end=",")
print()
for i in range(55):
  print(ord(BAA[i])-BAA_result[i]+i, end=",")

'''
24,18,18,18,18,18,18,18,18,18,34,28,28,28,28,28,24,18,18,18,18,18,18,18,18,18,34,28,28,28,28,28,24,18,18,18,18,18,18,18,18,18,34,28,28,28,28,28,24,18,18,18,18,18,18, 6,6,6,6,6,6,0,0,0,0,16,16,16,16,16,16,6,6,6,6,6,6,0,0,0,0,16,16,16,16,16,16,6,6,6,6,6,6,160,160,160,160,16,16,16,16,16,16,166,166,166,166,166,166,160, 24,24,24,24,24,24,24,24,18,18,34,34,34,34,34,34,24,24,24,24,24,24,24,24,18,18,34,34,34,34,34,34,24,24,24,24,24,24,24,24,18,18,34,34,34,34,34,34,24,24,24,24,24,24,24, 12,6,12,12,18,12,12,6,6,12,28,28,28,28,22,16,12,12,6,12,12,18,12,6,6,12,22,16,22,22,28,22,12,6,18,18,18,6,6,12,6,6,28,22,58,58,22,58,88,18,12,166,12,18,12, 42,42,36,36,36,36,36,36,36,36,52,52,46,46,46,46,42,42,36,36,36,36,36,36,36,36,52,52,52,46,46,46,42,42,42,36,36,36,36,36,36,36,52,52,52,46,46,46,42,42,42,36,36,36,36,
'''
```

## brute force
I decided to solve this via char by char brute force, because when I played around with it I discovered that chars don't affect their neighbours, ie some sort of xor keystream I suppose.

- I tried using `winappdbg` but Python 2 is dumb and won't work.
- I tried using `frida` but it just wouldn't attach itself to the program.
- I tried writing `Idascript` but they don't allow me to write to stdin.

Out of choices I patched the `exe` to print out the processed string.

Replacing `memcmp` with `cs:puts` which actually was called conveniently right below this part.

```diff
--- original @ 0x4E120D
-  E8 01 16 00 00        call    memcmp
+  FF 15 B5 1F 00 00     call    qword ptr [RIP+0x1FB5]
```

Then I `nop` out rest of the line.

```
.text:00000000004E120D                 call    cs:puts
.text:00000000004E1213                 nop
.text:00000000004E1214                 nop
.text:00000000004E1215                 nop
```

It works (IDA shows it).

Now the brute force solver.

```python
import string
from pwn import *
context.log_level = 'error'

ANS = [0, 33, 34, 4, 119, 82, 48, 17, 112, 41, 71, 1, 102, 113, 97, 52, 103, 19, 101, 51, 97, 85, 36, 35, 36, 35, 87, 117,
       120, 19, 105, 129, 36, 80, 105, 115, 151, 114, 135, 48, 117, 64, 52, 104, 87, 134, 151, 41, 136, 84, 100, 87, 0, 56, 145]
ANS_NOZ = [i if i != 0 else 1 for i in ANS]
FLAG_LEN = len(ANS_NOZ)
charset = string.printable.replace('\n', '').replace('\r', '')

found = ''
for i in range(FLAG_LEN):
    for c in charset:
        trial = found + c + 'A' * (FLAG_LEN - len(found) - 1)
        p = process('./void.exe')
        p.sendline(b"UMASS{" + trial.encode() + b"}")
        data = p.recvall(timeout=0.1)
        res = data.split(b"\r\n")[-3]
        if res[:i+1] == bytes(ANS_NOZ[:i+1]):
            found += c
            print(f"Position {i}: found '{c}' -> {found}")
            p.close()
            break
        p.close()
    else:
        raise Exception(f"No match found at position {i}")

print("Recovered flag: UMASS{" + found + "}")
result = ''
for i in range(len(found)):
    if ANS[i] == 0:
        result += chr(ord(found[i]) ^ 1)
    else:
        result += found[i]
print("Flag with XOR: UMASS{" + result + "}")
```

## flag

```flag
UMASS{0DD1y_H4nD_0ptiMi2eD_X8664_pr0gr4M_by_m3_;>_Soy4jGPHr3g}
```

Yay.