---
ai_date: 2025-04-27 05:19:35
ai_summary: Reversed EVM code reveals multiplication check, flag is derived from the result
ai_tags:
  - decomp
  - math
  - hex
created: 2024-08-04T06:14
points: 363
solves: 294
tags:
  - web3
updated: 2025-07-14T09:46
---

Just decompile the EVM byte code. [app.dedaub.com/decompile](https://app.dedaub.com/decompile)

`5f346113370265fdc29ff358a314601257ff00`

```asm
    0x0: PUSH0     
    0x1: CALLVALUE 
    0x2: PUSH2     0x1337
    0x5: MUL       
    0x6: PUSH6     0xfdc29ff358a3
    0xd: EQ        
    0xe: PUSH1     0x12
   0x10: JUMPI     
   0x11: SELFDESTRUCT
   0x12: STOP
```

A value is multiplied with `0x1337` and checked for equality against `0xfdc29ff358a3`.

The answer is hence `0xfdc29ff358a3/0x1337=0xd34db33f5`

```flag
n00bz{0xd34db33f5}
```