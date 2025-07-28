---
ai_date: 2025-04-27 05:25:40
ai_summary: Challenge involves exploiting bash history to bypass permissions and access flag (sudo required)
ai_tags:
  - bash
  - history
  - sudo
created: 2025-03-01T22:28
points: 150
solves: 17
updated: 2025-07-14T09:46
---

??? IDK What the task is even about, we have `sudo` right away and the flag is just right there.

What is detach, attach about???

```bash
ned@ctfd-chall2025-detached-then-forgotten-cz5r2:~$ cd ..
ned@ctfd-chall2025-detached-then-forgotten-cz5r2:/home$ ls
ned  ned-stash  ubuntu
ned@ctfd-chall2025-detached-then-forgotten-cz5r2:/home$ cd ned-stash/
-bash: cd: ned-stash/: Permission denied
ned@ctfd-chall2025-detached-then-forgotten-cz5r2:/home$ sudo ls ned-stash/
flag.txt
ned@ctfd-chall2025-detached-then-forgotten-cz5r2:/home$ sudo cat ned-stash/flag.txt
ATHACKCTF{t0ld_y4_n0t_to_b3_f00led_by_bash_hist0ry}ned@ctfd-chall2025-detached-then-forgotten-cz5r2:/home$
```

```flag
ATHACKCTF{t0ld_y4_n0t_to_b3_f00led_by_bash_hist0ry}
```