---
created: 2024-08-04T06:09
updated: 2024-08-05T19:41
points: 343
solves: 315
tags:
  - brainfuck
---

Using [Brainfuck Debugger (bxt.gitlab.io)](https://bxt.gitlab.io/brainfuck-debugger/) we can observe that the code increments `r0` to some value and decrements it to 0 quite a few times (`<[-]>`).

We can add a `<.>` before every decrement to 0 operation to output the values.

And it works.

```bf
>+++++++++++[<++++++++++>-]<.><[-]>
++++++++[<++++++>-]<.><[-]>
++++++++[<++++++>-]<.><[-]>
++++++++++++++[<+++++++>-]<.><[-]>
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++[<++>-]<.><[-]>
+++++++++++++++++++++++++++++++++++++++++[<+++>-]<.><[-]>
+++++++[<+++++++>-]<.><[-]>
+++++++++++++++++++[<+++++>-]<.><[-]>
+++++++++++[<+++++++++>-]<.><[-]>
+++++++++++++[<++++>-]<.><[-]>
+++++++++++[<++++++++++>-]<.><[-]>
+++++++++++++++++++[<+++++>-]<.><[-]>
+++++++++++[<+++++++++>-]<.><[-]>
++++++++[<++++++>-]<.><[-]>
++++++++++[<++++++++++>-]<.><[-]>
+++++++++++++++++[<+++>-]<.><[-]>
+++++++++++++++++++[<+++++>-]<.><[-]>
+++++++[<+++++++>-]<.><[-]>
+++++++++++[<++++++++++>-]<.><[-]>
+++++++++++++++++++[<+++++>-]<.><[-]>
++++++++++++++[<+++++++>-]<.><[-]>
+++++++++++++++++++[<++++++>-]<.><[-]>
+++++++++++++[<++++>-]<.><[-]>
+++++++[<+++++++>-]<.><[-]>
+++++++++++[<++++++++++>-]<.><[-]>
+++++++++++++++++[<++++++>-]<.><[-]>
+++++++[<++++++>-]<.><[-]>
+++++++++++[<+++++++++>-]<.><[-]>
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++[<+>-]<.><[-]>
+++++++++++[<+++>-]<.><[-]>
+++++++++++++++++++++++++[<+++++>-]<.><[-]
```

```flag
n00bz{1_c4n_c0d3_1n_br41nf*ck!}
```