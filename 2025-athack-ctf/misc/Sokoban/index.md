---
ai_date: 2025-04-27 05:25:20
ai_summary: Exploited Sokoban logic to solve puzzle and gain access to flag
ai_tags:
  - sokoban
  - puzzle
  - logic
created: 2025-03-02T04:34
points: 150
solves: 9
updated: 2025-07-14T09:46
---

Solved it with [sokoban-solver](https://github.com/dangarfield/sokoban-solver).

```python
import socket
from solver import solveSokodan
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("127.0.0.1", 5082))
s.recv(1024).decode()
s.send(b'\n')
while True:
    board = s.recv(1024).decode().replace('$', 'B').replace('@', '&')
    if '&' not in board:
        break
    sol, time = solveSokodan('astar', board.split('\n'))
    s.send(sol.encode())
    print(s.recv(1024).decode())
print(board)
```

```flag
ATHACKCTF{G0dIHateTh3s3StuFF}
```