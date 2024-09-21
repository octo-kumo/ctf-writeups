---
created: 2024-09-07T13:52
updated: 2024-09-20T23:55
solves: 11
points: 200
---

From the response of `/play` we can infer that this is a websocket endpoint.

```
< HTTP/2 400 
< date: Sat, 07 Sep 2024 17:47:31 GMT
< content-type: text/plain; charset=utf-8
< content-length: 12
< sec-websocket-version: 13
< x-content-type-options: nosniff
< strict-transport-security: max-age=15724800; includeSubDomains
< server: ArvanCloud
< server-timing: total;dur=400
< x-cache: BYPASS
< x-request-id: 1d4fe38ebe5143995cbeb137d1146d98
< x-sid: 6230
```

And it indeed is.

```
Welcome to Bingo!
You can get the list of commands by sending help command.
```

After some tinkering I decided to just go with it and play the game.

```js
import WebSocket from 'ws';
class BingoBoard {
    constructor(inputStr) {
        this.board = [];
        this.marked = Array(5).fill(null).map(() => Array(5).fill(false));
        this.cols = ['B', 'I', 'N', 'G', 'O'];
        const rows = inputStr.trim().split('\n');
        for (let i = 1; i < rows.length; i++) {
            const numbers = rows[i].trim().split(/\s+/);
            this.board.push(numbers.map(num => (num === 'X' ? 'X' : parseInt(num, 10))));
        }
    }

    check(str) {
        const match = str.match(/^([BINGO]):\s*(\d+)$/);
        if (!match) {
            return false;
        }
        const [_, col, number] = match;
        const colIndex = this.cols.indexOf(col);
        const num = parseInt(number, 10);
        for (let row = 0; row < 5; row++) {
            if (this.board[row][colIndex] === num) {
                this.marked[row][colIndex] = true;
                return true;
            }
        }
        return false;
    }

    checkBingo() {
        for (let row = 0; row < 5; row++)
            if (this.marked[row].every(mark => mark))
                return true;
        for (let col = 0; col < 5; col++)
            if (this.marked.every(row => row[col]))
                return true;
        if (this.marked.every((row, i) => row[i])) return true;
        if (this.marked.every((row, i) => row[4 - i])) return true;
        return false;
    }

    printBoard() {
        let str = "";
        for (let row = 0; row < 5; row++) {
            for (let col = 0; col < 5; col++) {
                str += this.marked[row][col] ? 'X' : this.board[row][col];
                str += ' ';
            }
            str += '\n';
        }
        return str;
    }
}
const socket = new WebSocket('https://keep-alive.uctf.ir/play');
let board = null;
socket.addEventListener('message', function (event) {
    let l = event.data;
    if (event.data.match(/B\s*I\s*N\s*/)) {
        board = new BingoBoard(event.data);
        l = "Board received.";
        socket.send("play");
    } else {
        if (board?.check(event.data)) {
            socket.send("mark");
            l += " :: on board"
            if (board.checkBingo()) {
                l += " :: BINGO!";
            }
        }
    }
    if (event.data.includes("Bingo! Your Flag")) {
        console.log(board?.printBoard());
    }
    console.log(l);
});
socket.addEventListener('open', function (event) {
    socket.send("get board");
});
```

```
Number marked
G: 49
I: 17
B: 4
B: 8
I: 28
N: 31
O: 65 :: on board :: BINGO!
Number marked
13 16 X 56 X
1 27 45 55 X
X 23 X 51 X
9 20 35 58 X
11 25 X X X

Bingo! Your Flag: UCTF{P1nk_Lak3_Maharl00}
Please reset the game with 'reset'
```

And oh, that was it.

```flag
UCTF{P1nk_Lak3_Maharl00}
```
