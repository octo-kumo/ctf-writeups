---
ai_date: 2025-04-27 05:25:28
ai_summary: Format string vulnerability in printf with user-supplied input
ai_tags:
  - fmt-str
created: 2025-03-01T22:32
tags:
  - unsolved
updated: 2025-07-14T09:46
---

Couldn't solve it, probably some fancy triple format string.

```c
__int64 __fastcall main(int a1, char **a2, char **a3)
{
  unsigned int v3; // eax
  int v4; // eax
  unsigned int v6; // [rsp+8h] [rbp-C28h] BYREF
  unsigned int v7; // [rsp+Ch] [rbp-C24h]
  char s[16]; // [rsp+10h] [rbp-C20h] BYREF
  char v9[1024]; // [rsp+20h] [rbp-C10h] BYREF
  char v10[1024]; // [rsp+420h] [rbp-810h] BYREF
  char format[1032]; // [rsp+820h] [rbp-410h] BYREF
  unsigned __int64 v12; // [rsp+C28h] [rbp-8h]

  v12 = __readfsqword(0x28u);
  v3 = time(0LL);
  srand(v3);
  printf("Before we start, type your username:\n username=");
  fgets(s, 16, stdin);
  s[strcspn(s, "\n")] = 0;
  sub_40128A();
  printf("Please enter your answer (a number):\n answer=");
  __isoc99_scanf("%d", &v6);
  v4 = rand();
  v7 = v4 + (v4 == -1);
  if ( v7 == v6 )
  {
    puts("Yes, you were right!");
  }
  else
  {
    snprintf(v9, 0x400uLL, "No %s! you are wrong!", s);
    snprintf(v10, 0x400uLL, "%s %s", v9, "The right answer was %d.");
    snprintf(format, 0x400uLL, "%s %s", v10, "Not %d.\n");
    printf(format, v7, v6);
  }
  printf("You were 100%% wrong. Bye!\n");
  return 0LL;
}
```