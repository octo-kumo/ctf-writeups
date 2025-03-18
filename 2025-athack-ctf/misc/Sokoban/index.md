---
created: 2025-03-02T04:34
updated: 2025-03-18T02:28
points: 150
solves: 9
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
