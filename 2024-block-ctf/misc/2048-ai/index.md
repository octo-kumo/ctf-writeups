---
ai_date: '2025-04-27 05:13:26'
ai_summary: Used Monte Carlo AI algorithm to solve 2048 game with 100% win rate, showcasing
  optimization and AI techniques.
ai_tags:
- ai
- monte-carlo
- game-theory
created: 2024-11-14T17:11
points: 150
solves: 31
title: 2048 ai
updated: 2024-11-14T17:33
---

I was working on this while @Maximxls solved it already 😭

## solve

I referenced this answer on stackoverflow.

> [What is the optimal algorithm for the game 2048?](https://stackoverflow.com/a/23853848/18196178)

> I found a simple yet surprisingly good playing algorithm: To determine the next move for a given board, the AI plays the game in memory using random moves until the game is over. This is done several times while keeping track of the end game score. Then the average end score per starting move is calculated. The starting move with the highest average end score is chosen as the next move.
> @[Ronenz](https://stackoverflow.com/users/632039/ronenz)

Then I yoinked his code from [ronzil/2048-ai-cpp: MonteCarlo AI for the 2048 game](https://github.com/ronzil/2048-ai-cpp), modified `RUNS` to be 100 and deeper runs to be 1000 since we aren't aiming for 8192.

I yoinked the controller from [2048-ai-cpp/2048.py at master · ronzil/2048-ai-cpp](https://github.com/ronzil/2048-ai-cpp/blob/master/2048.py) with slight modifications (a custom game controller)

```python [solve.py]
class CameControl:
    def __init__(self):
        self.proc = None
        self.restart()

    def restart(self):
        if self.proc != None:
            self.proc.close()

        self.proc = remote('54.85.45.101', 8006)
        self.board = None
        self.score = 0
        self.status = "running"
        self.update()

    def update(self):
        board = self.proc.recvuntil(b"\n\n", drop=True)
        self.board = [list(map(_from_val, row.split())) for row in board.decode().strip().split("\n")]
        status = self.proc.recv().decode().strip()
        if 'Enter command' in status:
            self.status = 'running'
        elif 'Game Over' in status:
            self.status = 'ended'
        else:
            self.status = 'won'
            print(status)

    def get_board(self):
        return self.board

    def get_status(self):
        return self.status

    def execute_move(self, move):
        self.proc.sendline(movekey(move))
        self.update()

    def get_highest(self):
        return max(max(row) for row in to_val(self.board))
```

### flag
I found it to run locally with a win rate of 100%, average time: 16.9s, and average moves: 988.

Great results.

```
...
Congratulations! You reached 2048 and won the game!
flag{f4st3r_th4n_hum4nly_p0ssibl3}
Game over. highest tile 2048.
Total time: 097.859959
Total moves: 1024
```

```flag
flag{f4st3r_th4n_hum4nly_p0ssibl3}
```