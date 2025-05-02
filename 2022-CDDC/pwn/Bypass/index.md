---
ai_date: '2025-04-27 05:11:19'
ai_summary: Password validation using hardcoded string comparison and input sanitization
ai_tags:
- str
- decomp
- xss-prevention
created: 2024-06-11T01:17
updated: 2024-07-07T23:07
---

By decompiling, we first try to figure out the login process

```cpp
if ((char)buf != '\0') {
    uVar3 = strlen(&buf);
    iVar1 = strncmp(&buf, *(undefined8 *)0x202010, uVar3);
    if (iVar1 == 0) {
        printf("Password : ");
        fflush(_stdout);
        read(0, &buf, 0x14);
        iVar2 = strcspn(&buf, 0x11a4);
        *(undefined *)((int64_t)&buf + iVar2) = 0;
        if ((char)buf != '\0') {
            uVar3 = strlen(&buf);
            iVar1 = strncmp(&buf, 0x202040, uVar3);
            if (iVar1 == 0) {
                *(undefined4 *)0x20203c = 1;
                goto code_r0x00000ec0;
            }
        }
        puts("[!] Wrong password!");
        *(undefined4 *)0x20203c = 0;
        goto code_r0x00000ec0;
    }
}
puts("\n[!] Wrong ID!");
```