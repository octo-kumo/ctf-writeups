---
ai_date: 2025-04-27 05:28:56
ai_summary: Executed `decrypt.sh` on each file in 'files' directory, filtered for lines containing 'picoCTF'
ai_tags:
  - shell
  - decrypt
  - grep
created: 2024-08-07T21:44
updated: 2025-07-14T09:46
---

```bash
for i in files/*; do decrypt.sh $i; done 2>/dev/null | grep pico
```

1. For every file under `files`, run `decrypt.sh`
2. `2>/dev/null` remove `stderr`
3. `grep pico` remove other lines than the flag

```flag
picoCTF{trust_but_verify_c6c8b911}
```