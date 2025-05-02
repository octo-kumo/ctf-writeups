---
ai_date: '2025-04-27 05:29:03'
ai_summary: Overflow and code execution to bypass level checks
ai_tags:
- overflow
- bypass
- level-checks
created: 2024-08-08T02:49
updated: 2024-08-08T04:38
---

Moving out of bounds is simple.
And we can have infinite lives by simply going backwards from the start of the grid.

Since player info is right next to the grid, we underflow the grid to get infinite lives.

Afterwards we can use `p` to solve the grid for us.

```
wwwaaaaaaaawssp
End tile position: 29 89
Lives left: 1073741861
```

But we get stuck on level 4.

---

Looked online and found the solution lol.

We have to somehow skip the level 4 check and jump to `0x08049970`.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1723105941/2024/08/bf88dc88535ec81d0427876365efe356.png)

Afterwards we have to skip the check again and jump to `0x080499fe`, because the value of wins is checked by the `win()` function.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1723105954/2024/08/07e61295a5c4f9664cc1e09f3d96bc7a.png)

```flag
picoCTF{gamer_leveluP_334c3e00}
```