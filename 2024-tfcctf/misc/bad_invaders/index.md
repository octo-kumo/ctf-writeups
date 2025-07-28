---
ai_date: 2025-04-27 05:21:20
ai_summary: Exploits ping feature to find bot's ID, then moves bot and shoots to win
ai_tags:
  - ping
  - id-enum
  - rce
created: 2024-08-02T18:24
points: 338
solves: 48
updated: 2025-07-14T09:46
---

The game goes like this.

1. Both players move with `PID|M|1;` (bot stops at position 3)
2. Game starts
3. Use `PID|S;` to shoot.
4. If both players are on the same position and that position is not 3, we win.

Obviously we have to move the bot away from 3 somehow.
To do this however we have to obtain the bot's id.

Retrying over and over again to hit a lucky id is not likely and probably very slow, so I will use the ping feature to find the bot's id.

We will ping every number until the server responds with `Pong`, ignore our own id, and we will get the bot's id.

We will then move our player and the bot by `1`, and then shoot.

## solve script

```python
import time
from pwn import *
context.log_level = "error"


def find_bot_id(s, pid):
    for i in range(100):
        s.send(f"{i}|P;".encode())
        data = s.recvuntil(b";", drop=True)
        if b'Pong' in data:
            print("ping found", i)
            if pid != i:
                return i


s = remote("challs.tfcctf.com", port=31994)
s.send(b"N|kumo;")
LAST_PING_TIME = time.time()
PLAYER_ID = -1
BOT_ID_GUESS = 69
while True:
    if time.time() - LAST_PING_TIME > 2 and PLAYER_ID != -1:
        s.send(f"{PLAYER_ID}|P;".encode())
        LAST_PING_TIME = time.time()
    data = s.recvuntil(b";", drop=True)
    print("received", data)
    if data[:1] == b"J":
        PLAYER_ID = int(data[2:])
        print("joined as", PLAYER_ID)
    if b'M|bot|' in data:
        print("bot moved!")
        s.send(f"{PLAYER_ID}|M|1;".encode())
    if b"G" in data:
        print("started!")
        bid = find_bot_id(s, PLAYER_ID)
        s.send(f"{PLAYER_ID}|M|1;".encode())
        s.send(f"{bid}|M|1;".encode())

        s.send(f"{PLAYER_ID}|S;".encode())
    if b'S|' in data:
        print("score", data)
```