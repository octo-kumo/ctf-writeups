---
ai_date: 2025-04-27 05:17:34
ai_summary: Found black dogs with blue collars and used `/data` command to get the flag from one of them.
ai_tags:
  - ssrf
  - exploitation
  - minecraft
created: 2024-07-20T02:55
points: 100
solves: 127
tags:
  - minecraft
updated: 2025-07-14T09:46
---

## minecraft
It's a Minecraft world!

Upon joining the world we are met with 1857 dogs, each with a different hex name!

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721458930/2024/07/a2d07430ae786c668a56f4da4f4bec1e.png)

In the challenge files, we can find a black dog with blue collar, I guess we have to find that dog.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721458966/2024/07/73e4dde4cc6002d91e6571f71f2e975c.png)

## solve
First we teleport all black dogs with blue collars to us.

```
/tp @e[type=minecraft:wolf,nbt={"variant": "minecraft:black","CollarColor":11b}] @p
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721458800/2024/07/f041638ef4f525e4b917fb9547d6ada4.png)

Hrm, there's two of them.

Then we use `/data` on the nearest dog to get its name. (use logs for easy copy paste)

```
/data get entity @e[type=minecraft:wolf,nbt={"variant": "minecraft:black","CollarColor":11b},limit=1,sort=nearest]
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721458692/2024/07/1239b25ef4f26aa0a35a0091db89568f.png)

...its not the correct flag, try the other one.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721458777/2024/07/68521dbfc4baab404ecfcb535977819c.png)

And we solved it.

```flag
ictf{6ed247d7539bb3bf}
```