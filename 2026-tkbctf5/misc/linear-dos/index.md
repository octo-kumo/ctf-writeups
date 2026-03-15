---
created: 2026-03-14T04:50
updated: 2026-03-15T01:14
points: 168
solves: 31
---

Was playing around and tweaking the regex, but then I realized I'm just doing manual optimization.

Instead wrote an optimizer to do it for me.

```js
import fs from "fs";
const MAX_TOTAL = 1800;

function measure(pattern) {
  let best = 0;
  let bestInputLen = 0;

  const maxInput = MAX_TOTAL - pattern.length;
  if (maxInput <= 0) return { score: -1 };

  for (let n = 10; n <= maxInput; n += Math.max(5, Math.floor(maxInput / 40))) {
    const input = "a".repeat(n);

    try {
      const re = new RegExp(pattern, "l");

      const start = process.hrtime.bigint();
      re.test(input);
      const end = process.hrtime.bigint();

      const ms = Number(end - start) / 1e6;

      if (ms > best) {
        best = ms;
        bestInputLen = n;
      }
    } catch {
      return { score: -1 };
    }
  }

  return { score: best, inputLen: bestInputLen };
}

function mutate(p) {
  const ops = [
    (x) => `(${x})*`,
    (x) => `(${x})+`,
    (x) => `(${x}|a)`,
    (x) => `(a|${x})`,
    (x) => `${x}|${x}`,
    (x) => `(${x}${x})`,
    (x) => `((${x})+)+`,
    (x) => `((${x})*)+`,
  ];

  const op = ops[Math.floor(Math.random() * ops.length)];
  return op(p);
}

function optimize(iter = 20000) {
  let bestPattern = "a|b";
  let bestScore = 0;
  let bestInput = 0;

  for (let i = 0; i < iter; i++) {
    let candidate = bestPattern;

    const mutCount = 1 + Math.floor(Math.random() * 3);
    for (let j = 0; j < mutCount; j++) {
      candidate = mutate(candidate);
    }

    if (candidate.length >= MAX_TOTAL) continue;

    const result = measure(candidate);

    if (result.score > bestScore) {
      bestScore = result.score;
      bestPattern = candidate;
      bestInput = result.inputLen;

      console.log(
        "New best:",
        bestScore.toFixed(2),
        "ms | input:",
        bestInput,
        "| pattern length:",
        bestPattern.length,
      );
      fs.writeFileSync(
        "best_pattern.json",
        JSON.stringify(
          {
            pattern: bestPattern,
            score: bestScore,
            input: "a".repeat(bestInput),
          },
          null,
          2,
        ),
      );
    }
  }

  return { bestPattern, bestScore, bestInput };
}

const res = optimize();

console.log("\nBest pattern:");
console.log(res.bestPattern);
console.log("Best input length:", res.bestInput);
console.log("Time:", res.bestScore, "ms");
```

```
❯ node --enable-experimental-regexp-engine opti.js
New best: 0.01 ms | input: 10 | pattern length: 8
New best: 2.70 ms | input: 1770 | pattern length: 26
New best: 29.85 ms | input: 1648 | pattern length: 113
New best: 62.29 ms | input: 1648 | pattern length: 119
New best: 65.03 ms | input: 1531 | pattern length: 239
New best: 129.68 ms | input: 1530 | pattern length: 246
New best: 1173.95 ms | input: 810 | pattern length: 990
New best: 1197.36 ms | input: 770 | pattern length: 998
New best: 1243.62 ms | input: 770 | pattern length: 1001
New best: 1321.14 ms | input: 789 | pattern length: 1008
```

This was enough for the remote machine.

```python
import json
import subprocess
from pwn import *


conn = remote("35.194.108.145", 57364)
conn.recvuntil(b"proof of work:")
cmd = conn.recvuntil(b"solution:", drop=True).strip().decode()

print(f"Solving PoW: {cmd}")
result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
print(f"PoW result: {result.stdout.strip()}")
conn.sendline(result.stdout.strip().encode())

data = json.load(open("best_pattern.json", "r"))
pattern = data["pattern"]
input_string = data["input"]

conn.sendline(pattern.encode())
conn.sendline(input_string.encode())

conn.interactive()
```

And done!

```flag
tkbctf{O(NM)dosu~}
```
