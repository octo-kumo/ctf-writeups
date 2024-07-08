---
created: 2024-06-11T01:17
updated: 2024-07-07T23:07
---

From the decompiled result we find two `strcmp`

```cpp
void func(void) {
    undefined4 uVar1;
    int32_t iVar2;
    undefined4 uVar3;
    undefined auStack144 [4];
    char *buf;
    
    printf(*(undefined4 *)0x8740);
    fflush(**(undefined4 **)0x8744);
    read(0, auStack144, 0x14);
    uVar3 = **(undefined4 **)0x8748;
    uVar1 = strlen(**(undefined4 **)0x8748);
    iVar2 = strncmp(auStack144, uVar3, uVar1);
    if (iVar2 != 0) {
        puts(*(undefined4 *)0x874c);
        bye();
    }
    printf(*(undefined4 *)0x8750);
    fflush(**(undefined4 **)0x8744);
    read(0, auStack144, 0x14);
    iVar2 = strncmp(auStack144, *(undefined4 *)0x8754, 10);
    if (iVar2 == 0) {
        spawn_shell();
    } else {
        puts(*(undefined4 *)0x8758);
    }
    bye();
    return;
}
```

The two addresses store plaintext `r00t` and `s3cr3t_k3y`, with them we can solve this problem

```shell
$ nc 13.215.173.140 7001
Login : r00t
Password : s3cr3t_k3y
ls
arm
flag
run
cat flag
CDDC22{R3versing_4rm_fun_AND_G00d!!}
```
