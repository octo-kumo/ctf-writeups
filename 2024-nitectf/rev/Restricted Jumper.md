---
ai_date: 2025-04-27 05:21:08
ai_summary: Exploited Unity game vulnerability with UnityExplorer mod for unlimited access and found flag in level 1
ai_tags:
  - cheat
  - exploit
  - level-traversal
created: 2024-12-15T07:05
points: 440
solves: 9
tags:
  - unity
updated: 2025-07-14T09:46
---

It's a unity game! So I immediately injected [sinai-dev/UnityExplorer](https://github.com/sinai-dev/UnityExplorer?tab=readme-ov-file)

It is basically a mod that allows me to cheat.

1. Download melon loader v0.5.x, and ask it to patch the game.
2. Download [UnityExplorer Mono](https://github.com/sinai-dev/UnityExplorer/releases/latest/download/UnityExplorer.MelonLoader.Mono.zip)
3. Paste all contents of the mod file into the game folder.
4. Open game, press `Z`.
## free cam yay

With unity explorer I can turn on free cam and fly right to the end, and there is no flag, welp.
It wouldn't be this easy would it?

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1734264454/2024/12/2b1f5249d1fe07eb662eac308938cf7a.png)

## portal

The little portal object in the scene is positioned left of the starting point, and turning off some components of the player allow us to enter it.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1734264506/2024/12/9f904825f2d3443b2eefde404ec29978.png)

## level 10

After entering the portal we are in level 10!

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1734264551/2024/12/b5db9e2f2f2e162a10173b9a4ff17d8a.png)

Nothing useful tho, then I realised there are 100 levels!
## the levels

Randomly, I realised that level 1 had the flag header, this means that the other levels probably contain rest of the flag parts.

Oh well, 😫 need to search through all the levels.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1734264600/2024/12/6d319cb52185c64d96d2473c51ae6fa3.png)

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1734264607/2024/12/889dd3e8546a4af0bb556b09f152e1ff.png)

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1734264613/2024/12/f5eb4d1ae4b23b4bed142008622dc21e.png)

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1734264621/2024/12/018f15b51fe11e43ed8ae0ba71df021b.png)

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1734264356/2024/12/c8a5e1bdd104323bc39631e039f51856.png)

```flag
nite{n3w_ar3a5_t0_f1nd_87862}
```

Coz the event was ending, I had to manually search all the levels. PAIN.