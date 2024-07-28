---
created: 2024-07-27T17:05
updated: 2024-07-28T02:14
title: GoLParty
---

> Welcome to the GoL Party!

Game of Life!

I've tried using wraparound grids but they don't really work, after quite a long time of trial and error, I realised that we are supposed to use a infinite grid.

I asked ChatGPT for an implementation and it works, nice.

This implementation works well because the correct answer is quite small (previously obtained via brute forcing for algo checking), only around ~300 live cells.

Hence, an implementation based on individual live cells would perform better than an actual 2D array with a lot of dead cells.
## Solve Script

```python
from pwn import *
import numpy as np
import sys

context.log_level = 'error'


class InfiniteGameOfLife:
    def __init__(self):
        self.live_cells = set()

    def add_live_cell(self, x, y):
        self.live_cells.add((x, y))

    def remove_live_cell(self, x, y):
        self.live_cells.discard((x, y))

    def get_neighbors(self, x, y):
        return [(x + dx, y + dy) for dx in range(-1, 2) for dy in range(-1, 2) if not (dx == 0 and dy == 0)]

    def count_live_neighbors(self, x, y):
        return sum((nx, ny) in self.live_cells for nx, ny in self.get_neighbors(x, y))

    def next_generation(self):
        new_live_cells = set()
        potential_cells = set()
        for cell in self.live_cells:
            potential_cells.add(cell)
            potential_cells.update(self.get_neighbors(cell[0], cell[1]))

        for cell in potential_cells:
            live_neighbors = self.count_live_neighbors(cell[0], cell[1])
            if cell in self.live_cells and live_neighbors in [2, 3]:
                new_live_cells.add(cell)
            elif cell not in self.live_cells and live_neighbors == 3:
                new_live_cells.add(cell)

        self.live_cells = new_live_cells

    def total_live_cells(self):
        return len(self.live_cells)


conn = remote('35.224.11.111', 32051)
conn.recvuntil(b'Game On :D\x1b[m\n\n')
while True:
    try:
        board = conn.recvuntil(b'\n\n[*] Enter', drop=True, timeout=1)
    except:
        print(conn.recvall())
        break
    board = board.replace(b'\xe2\x96\xa0', b'#')
    conn.recvuntil(b'live cells after ')
    itercount = conn.recvuntil(b' ', drop=True)
    conn.recvuntil(b'generations: ')

    itercount = int(itercount.decode())
    board = np.array([[1 if c == ord('#') else 0 for c in r] for r in board.strip().split(b"\n")])
    game = InfiniteGameOfLife()
    for x in range(35):
        for y in range(70):
            if board[x, y] == 1:
                game.add_live_cell(x, y)

    for i in range(itercount):
        game.next_generation()
    conn.sendline(str(game.total_live_cells()).encode())
    conn.recvuntil(b'[+] Correct!\n')
    print(f"correct! {itercount=} {game.total_live_cells()=}")
conn.close()
sys.exit()
# b'[*] Game Over! Thank you for playing.\n[+] Congratulations! Here is your flag: DEAD{GoL_P4rty_W4s_4_Fun_4nd_1ntr1gu1ng_G4m3}'
```

```flag
DEAD{GoL_P4rty_W4s_4_Fun_4nd_1ntr1gu1ng_G4m3}
```
