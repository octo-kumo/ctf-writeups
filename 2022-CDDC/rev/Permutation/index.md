---
created: 2024-06-11T01:17
updated: 2024-07-07T23:08
---

Sending the exe to a decompiler gives us this

```cpp
void main(undefined8 param_1, int64_t param_2, int64_t param_3, int64_t param_4)
{
    int32_t iVar1;
    undefined8 uVar2;
    int64_t iVar3;
    char *pcVar4;
    undefined *puVar5;
    uint64_t arg4;
    undefined auStack200 [32];
    uint8_t uStack168;
    HMODULE pvStack160;
    char acStack152 [64];
    undefined auStack88 [64];
    uint64_t uStack24;
    
    uStack24 = *(uint64_t *)0x140004070 ^ (uint64_t)auStack200;
    pcVar4 = acStack152;
    for (iVar3 = 0x40; iVar3 != 0; iVar3 = iVar3 + -1) {
        *pcVar4 = '\0';
        pcVar4 = pcVar4 + 1;
    }
    puVar5 = auStack88;
    for (iVar3 = 0x40; iVar3 != 0; iVar3 = iVar3 + -1) {
        *puVar5 = 0;
        puVar5 = puVar5 + 1;
    }
    fcn.140001320((int64_t)"[+] Input flag\n> ", param_2, param_3, param_4);
    uVar2 = (*___acrt_iob_func)(0);
    (*_fgets)(acStack152, 0x3f, uVar2);
    uStack168 = sub.api_ms_win_crt_string_l1_1_0.dll_strlen(acStack152);
    pvStack160 = (HMODULE)(*_GetModuleHandleW)(0);
    arg4 = (uint64_t)uStack168;
    puVar5 = auStack88;
    pcVar4 = acStack152;
    fcn.140001060(pvStack160, pcVar4, (int64_t)puVar5);
    if (uStack168 == 0x2d) {
        puVar5 = (undefined *)0x2d;
        pcVar4 = (char *)0x140004000;
        iVar1 = sub.VCRUNTIME140.dll_memcmp(auStack88);
        if (iVar1 == 0) {
            fcn.140001320((int64_t)"[+] Good!\n", (int64_t)pcVar4, (int64_t)puVar5, arg4);
            goto code_r0x0001400012f8;
        }
    }
    fcn.140001320((int64_t)"[!] Retry!\n", (int64_t)pcVar4, (int64_t)puVar5, arg4);
code_r0x0001400012f8:
    fcn.140001410(uStack24 ^ (uint64_t)auStack200);
    return;
}
```

The flag length is `0x2d` or 45
