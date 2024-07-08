---
created: 2024-06-11T01:17
updated: 2024-07-07T23:08
---

By sending the file to a decompiler we get

```cpp
void func(char **arg3, int32_t arg1, int32_t arg_5ch, int32_t arg_1ch, int32_t arg_10h, int32_t arg_18h, int32_t arg_60h) {
    uint32_t uVar1;
    uint32_t uStack80;
    uint8_t auStack76 [64];
    int32_t iStack12;
    
    iStack12 = **(int32_t **)0x4110b4;
    (**(code **)0x411098)(auStack76, 0, 0x40);
    uStack80 = 0;
    while( true ) {
    // obj.flag
        uVar1 = (**(code **)0x4110a8)(0x411010);
        if (uVar1 <= uStack80) break;
    // obj.flag
        auStack76[uStack80] = "EBBE44}7YJip5YAn7bt2Yk7vuYtcp5tuohax{"[uStack80] ^ 6;
        uStack80 = uStack80 + 1;
    }
    // str.The_flag_is__s
    (**(code **)0x4110b8)(0x400cc0, auStack76);
    if (iStack12 != **(int32_t **)0x4110b4) {
        (**(code **)0x4110b0)();
    }
    return;
}
```

As we can see all the characters are XOR-ed with 6

And since XOR is reversible

```text
a ^ b = c
c ^ b = a
```

We can just repeat the process and get our flag

```text
CDDC22{1_Lov3_Gh1dr4_m1ps_rev3rsing~}
```
