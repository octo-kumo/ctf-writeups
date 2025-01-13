---
created: 2025-01-10T23:40
updated: 2025-01-12T19:00
solves: 323
points: 100
---

Not allowed to read files with `flag` in their name? Solved simply by making a symlink to the flag.

```c
if (strstr(f, "flag") != NULL)
{
    printf("Can't read the 'flag' file.\n");
    return 1;
}
```

```bash
user@containerssh-bxv6f:/challenge$ ln -s /flag.txt ~/fl  
user@containerssh-bxv6f:/challenge$ ./chal
Enter file to read: /home/user/fl
uoftctf{r4c3_c0nd1t10n5_4r3_c00l}
�z�
```

```flag
uoftctf{r4c3_c0nd1t10n5_4r3_c00l}
```