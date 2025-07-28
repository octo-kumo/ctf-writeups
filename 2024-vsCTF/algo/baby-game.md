---
ai_date: 2025-04-27 05:22:28
ai_summary: "Winning strategy based on Manhattan distance: if combined moves are even, second player wins; otherwise, first player wins."
ai_tags:
  - manhattan-distance
  - parity
  - game-theory
created: 2024-06-14T14:37
points: 272
solves: 345
updated: 2025-07-14T09:46
---

> 1. Players start at different positions on the grid.
> 2. In each round, one player makes a move. They can move exactly one square in one of the four directions: up, down, left, or right. Moving out of the battleground is forbidden.
> 3. A player wins the game by moving their character to the square occupied by the opponent’s character.
> 4. Both players are highly skilled: when they can win, they will win as soon as possible. When they can only lose, they will try to delay their loss as long as possible.
> Your task is to determine who will win the game, given n and their initial positions.

If we assume the game has ended, and flatten out their movement histories, we can see that the total number of tiles moved, combined, has the same even-ness, as the Manhattan distance between the two players.

That is, if the Manhattan distance is even, the second player would always, take the last winning step, vice versa.

```cpp
#include <iostream>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    
    int n;
    cin >> n;
    
    int x1, y1, x2, y2;
    cin >> x1 >> y1 >> x2 >> y2;
    
    // Calculate the Manhattan distance components
    int dx = abs(x2 - x1);
    int dy = abs(y2 - y1);
    
    // Determine the winner
    if ((dx + dy) % 2 == 1) {
        cout << "Caring Koala\n";
    } else {
        cout << "Red Panda\n";
    }

    return 0;
}

```