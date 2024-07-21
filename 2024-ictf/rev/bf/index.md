---
created: 2024-07-19T20:57
updated: 2024-07-21T16:08
solves: 225
points: 100
---

Using the debugger [Brainfuck Debugger (bxt.gitlab.io)](https://bxt.gitlab.io/brainfuck-debugger/), we can observe that each character is checked individually.

If the character is not correct the program will be stuck in an infinite loop.

Using python with timeout didn't work well.

So I used a JS library that allows me to set a maximum step limit.

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
