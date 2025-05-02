---
ai_date: '2025-04-27 05:22:25'
ai_summary: Koala wins by reaching d=0, while Panda wins by advancing when d is odd
  or when adjacent cells are decreasing in value
ai_tags:
- game-theory
- advantage
- manhattan-distance
created: 2024-06-15T00:52
updated: 2024-08-04T19:44
---

> In each round, one player makes a move. Caring Koala can move exactly one square in one of the four directions: up, down, left, or right. On the other hand, Red Panda can move one or two squares. Moving out of the battleground is forbidden

Ok now player 2 can move 1 or 2 tiles in one direction.

## Koala (move 1)

- $d=1$, win to $d=0$.
- $d=2$, evade to $d=3$.
- $d=3$, evade to $d=4$ if same row/col, otherwise advance to $d=2$.
- $d=4$, advance to $d=3$.
- $d=5$, advance to $d=4$
## Panda (move 2)

Manhattan distances $d$.

$$
\begin{cases}
\text{advance} & \text{if } d \text{ is odd} \\
a[i - 1] > a[i] & \text{if } d \text{ is even}
\end{cases} 
$$

Idk.