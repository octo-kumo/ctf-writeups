---
ai_date: 2025-04-27 05:18:03
ai_summary: "Solved by iterating through printable characters and checking if the Brainfuck program halts, revealing the flag: ictf{1_h4t3_3s0l4ng5_7d4f3a1b}"
ai_tags:
  - brute
  - bf
  - halt-problem
created: 2024-07-19T20:57
points: 100
solves: 225
tags:
  - brainfuck
updated: 2025-07-14T09:46
---

Using the debugger [Brainfuck Debugger (bxt.gitlab.io)](https://bxt.gitlab.io/brainfuck-debugger/), we can observe that each character is checked individually.

If the character is not correct the program will be stuck in an infinite loop.
## halting problem

We can hence solve it by checking each character individually, against every printable character, if the program finishes, it is the correct character.

Using python with timeout didn't work well.

So I used a JS library that allows me to set a maximum step limit.

## solve script

```js
import { Machine } from 'braincrunch';
import { readFileSync } from 'fs';
import { join } from 'path';
const ALL = readFileSync(join(import.meta.dirname, 'bf.txt'), 'utf8');
const codes = ALL.split(',').filter(Boolean).map(s => ',' + s)
const printables = Array.from({ length: 95 }, (_, i) => String.fromCharCode(i + 32));
let flag = "";
for (let code of codes) {
    for (let read of printables) {
        var machine = new Machine({ code, read });
        let ran = machine.run(5000);
        if (ran !== 5000) {
            flag += read;
            console.log(flag)
            break
        }
    }
}
// ictf{1_h4t3_3s0l4ng5_7d4f3a1b}
```

```flag
ictf{1_h4t3_3s0l4ng5_7d4f3a1b}
```