---
created: 2024-07-20T17:40
updated: 2024-07-21T16:07
tags:
  - algo
  - maze
solves: 30
points: 428
---

It's a maze.
A 40 by 40, 2D maze using filled or empty cells.

## interface

```python
self.set(*([0]*self.dim), val='@')           # starts  at 0,  0
self.set(*([self.size-1]*self.dim), val='F') # flag is at 39, 39
```

Interface to remote can be done with pwntools, using a timeout of 0.3s.
If server replies within 0.3 seconds it would either be a failure or we have reached the flag.

> I measured the response time to be around 0.08s, so I multiplied it by 4 to be safe.

```python
def rem_move(mv):
    key = list(moveDict.keys())[list(moveDict.values()).index(mv)]
    if key == None:
        print("invalid move")
        exit(1)
    conn.sendline(key.encode())
    r = conn.recvline(timeout=0.3)
    if b'BONK' in r:
        return False
    if b'ictf' in r:
        print(r)
    return True
```

## path find
I decided to use A* for path finding.

Starting off with a "grey" grid of unknown types, every time the bot moves, it will be able to determine the actual value of the grid it is moving towards.

Every time the bot encounters a wall, i.e. a grey tile turning into a black tile, blocking the bot's planned path, it will rebuild its path with A*.
### example
At the current location, the bot's planned path is downwards, but as we can see on the left (truth), that path is not possible.
![frame_0177.jpg](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721521632/2024/07/4e8a4bd8c7ac0d8aabfed8c1bd536697.jpg)

It discovers that, changing the grey tile to black to represent a wall, and executes a new round of A* pathfinding, this create a new path that now takes into account the updated grid.

![frame_0178.jpg](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721521689/2024/07/bac4f512777b0c3517a99506ebcd939e.jpg)

Once again, bumped into a wall. With the new information that it now has, the old path is completely blocked hence the new path.

![frame_0179.jpg](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721521767/2024/07/46e7799702bd354f61feef28a5e48acf.jpg)

## result
All attempts take around 200-1000 moves, with 0.3s timeout that evaluates to a 1 to 5 minute wait.
I believe that is pretty fast, and probably does not backtrack as much as BFS or DFS.

![output-remote.gif](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721521939/2024/07/3cf7a8fd7a84cbc571195a8aa6e6135b.gif)

Here is the local version for reference, a particular one with very bad luck.
![output-local.gif](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721521947/2024/07/009bbadedfb547f347626d504f67e356.gif)

## solve script

```python [solver.py]
import sys
from recorder import PygameRecord
from astar import a_star_search
from maze import Maze
import pygame
LOCAL = False

if not LOCAL:
    from pwn import *
    context.log_level = 'error'
    conn = remote('left-in-the-dark.chal.imaginaryctf.org', 1337)
    conn.recvuntil(b'Find the flag in this maze. Good luck!\r\nWASD to move.\r\n')

moveDict = {
    'w': (-1, 0),
    's': (1, 0),
    'd': (0, 1),
    'a': (0, -1),
}


def rem_move(mv):
    key = list(moveDict.keys())[list(moveDict.values()).index(mv)]
    if key == None:
        print("invalid move")
        exit(1)
    conn.sendline(key.encode())
    r = conn.recvline(timeout=0.3)
    if b'BONK' in r:
        return False
    if b'ictf' in r:
        print(r)
    return True


WIDTH, HEIGHT = 800, 400
GRID_SIZE = 40
CELL_SIZE = HEIGHT // GRID_SIZE

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption('Maze Exploration')

COLOR_MAP = {
    ' ': (255, 255, 255),
    '#': (0, 0, 0),
    '?': (128, 128, 128),
    'F': (0, 0, 255),
    '@': (0, 255, 0)
}


def draw_grid(start_x, start_y, grid, colormap):
    for x in range(GRID_SIZE):
        for y in range(GRID_SIZE):
            pygame.draw.rect(screen, colormap[grid[y][x]], (start_x + x*CELL_SIZE, start_y + y*CELL_SIZE, CELL_SIZE, CELL_SIZE), 0)


def draw_path(start_x, start_y, path):
    path_pixels = [(start_x + y*CELL_SIZE + CELL_SIZE//2, start_y + x*CELL_SIZE + CELL_SIZE//2) for (x, y) in path]
    if path_pixels:
        pygame.draw.lines(screen, (255, 0, 0), False, path_pixels, 2)
#####################


pos = (0, 0)
maze = Maze(2, 40)
flag = (39, 39)
mem_grid = [['?' for i in range(40)] for i in range(40)]
mem_grid[0][0] = ' '

total_moves = 0


def attempt_move(dest):
    global pos, total_moves
    diff = (dest[0]-pos[0], dest[1]-pos[1])
    print(pos, dest, diff)
    if abs(diff[0])+abs(diff[1]) != 1:
        print(f"serious error with path finding! {pos=} {dest=}")
        exit(1)
    total_moves += 1
    if LOCAL:
        result = maze.move(diff)
    else:
        result = rem_move(diff)
    if result:
        pos = (pos[0]+diff[0], pos[1]+diff[1])
        mem_grid[pos[0]][pos[1]] = ' '
        return True
    mem_grid[dest[0]][dest[1]] = '#'
    return False


clock = pygame.time.Clock()
running = True
pygame.font.init()
font = pygame.font.SysFont('Arial', 30)
path = None
victory_frames = 90
with PygameRecord("output-"+('local' if LOCAL else 'remote')+".gif", 30) as recorder:
    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

        screen.fill((255, 255, 255))
        if LOCAL:
            draw_grid(0, 0, maze.maze, COLOR_MAP)
        else:
            pygame.draw.rect(screen, (0, 0, 0), (0, 0, WIDTH//2, HEIGHT), 0)
        draw_grid(WIDTH//2, 0, mem_grid, COLOR_MAP)
        if pos != flag:
            if path == None:
                path = a_star_search(mem_grid, pos, flag)
                if path == None:
                    continue
            path[0] = pos
            draw_path(WIDTH//2, 0, path)
            p = path.pop(1)
            if not attempt_move(p):
                path = None
        else:
            victory_frames -= 1
            if victory_frames <= 0:
                running = False
        text_surface = font.render('total moves = '+str(total_moves)+" "+('won!' if pos == flag else ''), False, (255, 0, 255))
        screen.blit(text_surface, (0, 0))
        recorder.add_frame()
        pygame.display.flip()
        clock.tick(1000)
    print("saving...")
    recorder.save()
pygame.quit()
```

### a*

*adapted from [A\* Search Algorithm - GeeksforGeeks](https://www.geeksforgeeks.org/a-search-algorithm/)*

```python [astar.py]
import math
import heapq

# Define the Cell class


class Cell:
    def __init__(self):
        self.parent_i = 0  # Parent cell's row index
        self.parent_j = 0  # Parent cell's column index
        self.f = float('inf')  # Total cost of the cell (g + h)
        self.g = float('inf')  # Cost from start to this cell
        self.h = 0  # Heuristic cost from this cell to destination


# Define the size of the grid
ROW = 40
COL = 40

# Check if a cell is valid (within the grid)


def is_valid(row, col):
    return (row >= 0) and (row < ROW) and (col >= 0) and (col < COL)

# Check if a cell is unblocked


def is_unblocked(grid, row, col):
    return grid[row][col] != "#"

# Check if a cell is the destination


def is_destination(row, col, dest):
    return row == dest[0] and col == dest[1]

# Calculate the heuristic value of a cell (Euclidean distance to destination)


def calculate_h_value(row, col, dest):
    return ((row - dest[0]) ** 2 + (col - dest[1]) ** 2) ** 0.5

# Trace the path from source to destination


def trace_path(cell_details, dest):
    path = []
    row = dest[0]
    col = dest[1]

    while not (cell_details[row][col].parent_i == row and cell_details[row][col].parent_j == col):
        path.append((row, col))
        temp_row = cell_details[row][col].parent_i
        temp_col = cell_details[row][col].parent_j
        row = temp_row
        col = temp_col

    path.append((row, col))
    path.reverse()
    return path

# Implement the A* search algorithm


def a_star_search(grid, src, dest):
    if not is_valid(src[0], src[1]) or not is_valid(dest[0], dest[1]):
        print("Source or destination is invalid")
        return

    if not is_unblocked(grid, src[0], src[1]) or not is_unblocked(grid, dest[0], dest[1]):
        print("Source or the destination is blocked")
        return

    if is_destination(src[0], src[1], dest):
        print("We are already at the destination")
        return

    closed_list = [[False for _ in range(COL)] for _ in range(ROW)]
    cell_details = [[Cell() for _ in range(COL)] for _ in range(ROW)]

    i = src[0]
    j = src[1]
    cell_details[i][j].f = 0
    cell_details[i][j].g = 0
    cell_details[i][j].h = 0
    cell_details[i][j].parent_i = i
    cell_details[i][j].parent_j = j

    open_list = []
    heapq.heappush(open_list, (0.0, i, j))
    found_dest = False

    while len(open_list) > 0:
        p = heapq.heappop(open_list)
        i = p[1]
        j = p[2]
        closed_list[i][j] = True
        directions = [(0, 1), (0, -1), (1, 0), (-1, 0)]  # , (1, 1), (1, -1), (-1, 1), (-1, -1)]
        for dir in directions:
            new_i = i + dir[0]
            new_j = j + dir[1]

            # If the successor is valid, unblocked, and not visited
            if is_valid(new_i, new_j) and is_unblocked(grid, new_i, new_j) and not closed_list[new_i][new_j]:
                # If the successor is the destination
                if is_destination(new_i, new_j, dest):
                    # Set the parent of the destination cell
                    cell_details[new_i][new_j].parent_i = i
                    cell_details[new_i][new_j].parent_j = j
                    # print("The destination cell is found")
                    # Trace and print the path from source to destination
                    found_dest = True
                    return trace_path(cell_details, dest)
                else:
                    # Calculate the new f, g, and h values
                    g_new = cell_details[i][j].g + 1.0
                    h_new = calculate_h_value(new_i, new_j, dest)
                    f_new = g_new + h_new

                    # If the cell is not in the open list or the new f value is smaller
                    if cell_details[new_i][new_j].f == float('inf') or cell_details[new_i][new_j].f > f_new:
                        # Add the cell to the open list
                        heapq.heappush(open_list, (f_new, new_i, new_j))
                        # Update the cell details
                        cell_details[new_i][new_j].f = f_new
                        cell_details[new_i][new_j].g = g_new
                        cell_details[new_i][new_j].h = h_new
                        cell_details[new_i][new_j].parent_i = i
                        cell_details[new_i][new_j].parent_j = j

    # If the destination is not found after visiting all cells
    if not found_dest:
        print("Failed to find the destination cell")
        print(src, dest)
        print(grid)
        exit(1)
```

### maze
modified for local debugging.

```python [maze.py]
...
def move(self, mv):
	newLoc = (self.loc[0] + mv[0], self.loc[1] + mv[1])
	if (
		newLoc[0] < 0 or newLoc[0] >= self.size or
		newLoc[1] < 0 or newLoc[1] >= self.size or
		self.get(*newLoc) == '#'
	):
		# print("BONK")
		return False
	if self.get(*newLoc) == 'F':
		print("You win!")
	self.set(*self.loc, val=' ')
	self.set(*newLoc, val='@')
	self.loc = newLoc
	return True
...
```
