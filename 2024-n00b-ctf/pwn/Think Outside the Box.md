---
ai_date: 2025-04-27 05:20:34
ai_summary: Exploited integer overflow to manipulate game state and win without moving
ai_tags:
  - int-ovf
  - game-exploit
  - logic-bug
created: 2024-08-04T06:10
points: 464
solves: 152
updated: 2025-07-14T09:46
---

IDK how I solved it but sending it `4294967294,4294967294` will cause the bot to not move at all for 2 turns.

```
Welcome to Tic-Tac-Toe! In order to get the flag, just win! The bot goes first!
 | _   | _   | _   |
 | --- | --- | --- |
 | _   | X   | _   |
 | --- | --- | --- |
 | _   | _   | _   |
Move: 4294967294,4294967294
 | _   | _   | _   |
 | --- | --- | --- |
 | _   | X   | _   |
 | --- | --- | --- |
 | _   | _   | _   |
Bot turn!
 | _   | _   | _   |
 | --- | --- | --- |
 | _   | X   | _   |
 | --- | --- | --- |
 | _   | _   | _   |
Move: 0,0
 | O   | _   | _   |
 | --- | --- | --- |
 | _   | X   | _   |
 | --- | --- | --- |
 | _   | _   | _   |
Bot turn!
 | O   | _   | _   |
 | --- | --- | --- |
 | _   | X   | _   |
 | --- | --- | --- |
 | _   | _   | _   |
Move: 1,0
 | O   | _   | _   |
 | --- | --- | --- |
 | O   | X   | _   |
 | --- | --- | --- |
 | _   | _   | _   |
Bot turn!
 | O   | _   | _   |
 | --- | --- | --- |
 | O   | X   | X   |
 | --- | --- | --- |
 | _   | _   | _   |
Move: 2,0
 | O   | _   | _   |
 | --- | --- | --- |
 | O   | X   | X   |
 | --- | --- | --- |
 | O   | _   | _   |
You won! Flag: n00bz{l173r4lly_0u7s1d3_7h3_b0x_L0L_a79d2818f608}
```

```flag
n00bz{l173r4lly_0u7s1d3_7h3_b0x_L0L_a79d2818f608}
```