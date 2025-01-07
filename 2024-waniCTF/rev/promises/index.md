---
created: 2024-06-22T19:23
updated: 2025-01-06T23:25
solves: 15
points: 373
tags:
  - fav
---

> JavaScript の Promise について勉強した。なんかいろいろできますね！
> I just learnt about JavaScript promises. They are a very powerful construct!

## Analysis

The script file is very huge, but has a clear pattern to it.

After some analysis of the problem we can make sense of all the statements.

### Getter Setter Part

```js
GETTER_NAME = new Promise((resolve => {
    SETTER_NAME = resolve;
    counter++;
    if (counter === 25e3) taskAfterIncre25e3()
}));
```

I call this the getter/setter part because it is.

`GETTER_NAME` is called to get the value of this item, and `SETTER_NAME` will resolve value to this promise, effectively setting the value of this promise.

### Logic Part

```js
(async () => SETTER_C( GETTER_A < GETTER_B ? GETTER_A : GETTER_B))();
```

After removing the `await`, this is just a event handler, after all dependencies of the setter, `GETTER_A` and `GETTER_B` have resolved, this logic will execute, while also resolving any other `GETTER` corresponding to the `SETTER`.

### Idea

Since every line of code, have dependencies $l_{dep}\ge 0$, I find the actual execution order by repeated marking lines as "ran".

After extracting the actual logic, I will do further analysis.

## Logic Extraction

### Parsing the script

```js
import fs from "fs";
import jsbeautifier from "js-beautify";
import { dirname, join } from "path";
import { fileURLToPath } from 'url';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
/*
new Promise((resolve => {
    eQXZhHVpfElEktxA = WNRMgnBCfwgabWRJ;
    incre++;
    if (incre === 25e3) taskAfterIncre25e3()
}));
            */
let code = fs.readFileSync(join(__dirname, "promise.js"), "utf8");
code = jsbeautifier.js_beautify(code)

code = code.replace(/yJpYftBCPjwGmzAd/g, 'counter')
code = code.replace(/lbarHjWBfcaCFsrw/g, 'counter2')
code = code.replace(/VXzWAkPODJDoQpyz/g, 'taskAfterIncre25e3')

const hooks = {};
const increPromiseRegex = /([a-zA-Z]{16}) = new Promise\(\(([a-zA-Z]{16}) => \{[\n\s]*([a-zA-Z]{16}) = \2;[\n\s]*counter\+\+;[\n\s]*if \(counter === 25e3\) taskAfterIncre25e3\(\)[\n\s]*\}\)\);/g;
code = code.replace(increPromiseRegex, (match, token1, token2, token3) => {
    hooks[token3] = token1; // await hook will be resolved by 
    return match//token1 + " = increPromise;"
});
const promisesRegex = /\s*([a-zA-Z]{16}) = increPromise;[\s\n]*/g;
const promises = [];
console.log("replacing promises...");
code = code.replace(promisesRegex, function (match, token) {
    promises.push(token);
    return match;
});
const dependency = []
console.log("constructing dependency list...");
const step = /\(async \(\) => ([a-zA-Z]{16})\((.+)\)\)\(\);?/g;
code.replace(step, (match, token1, token2) => {
    const dep = {
        id: hooks[token1],
        dep: [...token2.matchAll(/await ([a-zA-Z]{16})/g)].map(a => a[1]),
        act: token2.replace(/await ([a-zA-Z]{16})/g, '$1')
            .replace(/^([a-zA-Z]{16}) < ([a-zA-Z]{16}) \? \1 : \2$/, 'min($1, $2)')
            .replace(/^([a-zA-Z]{16}) < ([a-zA-Z]{16}) \? \2 : \1$/, 'max($1, $2)')
    };
    if (dep.act.includes("min") || dep.act.includes("max")) {
        dep.tryCollapse = true;
        if (dep.dep.length < 2) {
            console.log("min/max operation with less than 2 dependencies")
        }
    }
    dependency.push(dep);
    return ``;
});
const last_step = /\(async \(\) => \{\n\s*(.+)\s*\}\)\(\);?/g;
code = code.replace(last_step, (match, content) => {
    dependency.push({
        id: "last step",
        dep: [...content.matchAll(/await ([a-zA-Z]{16})/g)].map(a => a[1]),
        act: content.replace(/await ([a-zA-Z]{16})/g, '$1')
    });
    return ``;
});
dependency.sort((a, b) => a.dep.length - b.dep.length);
dependency.forEach((dep) => {
    dep.triggers = dependency.filter((d) => d.dep.includes(dep.id)).map(d => d.id);
});
fs.writeFileSync(join(__dirname, "dependency.json"), JSON.stringify(dependency, null, 2));

code = jsbeautifier.js_beautify(code)
fs.writeFileSync(join(__dirname, "promises.new.js"), code);

console.log("Done");
```

This long piece of code may not be optimized nor short, but it handles the funny huge js file like a champ.

This produces a json file like this.

- `id` name of the getter, i.e. name of the promise
- `dep` dependencies, all dependencies must finish running before this logic can execute
- `act` the logic itself
- `triggers` reverse of `dep`, piece of code should trigger an update on them on completion

```json
{
    "id": "skAlFFvSKVzvPGqp",
    "dep": [
      "ptuzLEyBowOzFEAW"
    ],
    "act": "ptuzLEyBowOzFEAW & 1n",
    "triggers": [
      "zGDACyINydkHbQmY"
    ]
},
{
    "id": "CpoveyBLbblSQKWU",
    "dep": [
      "IWdzmMezgIrPfPAR"
    ],
    "act": "IWdzmMezgIrPfPAR >> 1n",
    "triggers": [
      "UFvIpYjlxWFeApbj",
      "ORwkQkzHpoIEGxMy"
    ]
},
  ```

### Sorting the operations

```js
import fs from "fs";
import { join, dirname } from "path";
import { fileURLToPath } from 'url';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
const operations = JSON.parse(fs.readFileSync(join(__dirname, "dependency.json"), "utf8"));
const operationsMap = {};
operations.forEach(op => operationsMap[op.id] = op);

const completed = {};
const translate = {};
const optimized = {};
const inOrder = [];
let i = 0;
console.log("Total operations:", operations.length);
console.log("Sorting operations...");

const shifterReg = /^(.+) >> (\d+)n$/;
/* WHILE THERE ARE STILL OPERATIONS LEFT TO DO */ 
while (operations.length > inOrder.length) {
	/* FIND ALL OPERATIONS THAT ARE INCOMPLETE & ALL DEPS ARE COMPLETED */
    const l = operations.filter(op => !completed[op.id] && op.dep.every(d => completed[d]));
    l.forEach(op => {
        // single dep optimization
        if (op.dep.length === 1) {
            let child = operationsMap[op.dep[0]].act;
            // if (child.endsWith(" & 1n")) child = `(${child})`;
            /* COLLAPSE >> 1n >> 1n CHAINS */
            if (child.match(shifterReg) && op.act.match(shifterReg) && op.act.endsWith(" >> 1n")) {
                (optimized[op.dep[0]] ??= []).push(op.id);
                op.act = child.replace(shifterReg, (match, act, n) => {
                    return `${act} >> ${parseInt(n) + 1}n`;
                });
            }
            // else op.act = op.act.replace(new RegExp(op.dep[0], "g"), child);
        }
		/* REPLACE ALL OCCURRENCES OF DEPENDENCIES WITH THEIR NEW NAME */
        op.dep.forEach(d => {
            // if (operationsMap[d].tryCollapse) {
            //     let child_act = operationsMap[d].act;
            //     if (child_act.substring(0, 4) === op.act.substring(0, 4)) {
            //         child_act = child_act.substring(4, child_act.length - 1);
            //     }
            //     op.act = op.act.replace(new RegExp(d, "g"), child_act);
            // }
            op.act = op.act.replace(new RegExp(d, "g"), translate[d]);
        });
        // if (op.tryCollapse) {
        //     // console.log(op.act)
        //     op.act = math.simplify(op.act).toString();
        //     return;
        // }
    });
    l.sort((a, b) => a.act.localeCompare(b.act));
    l.forEach(op => {
	    /* ASSIGN NEW NAME */
        translate[op.id] = "a" + (i++);
        completed[op.id] = true;
        inOrder.push(op);
    });
    if (l.length === 0) {
        console.log("No more operations can be completed");
        break;
    }
}

/* CHECK FOR INCOMPLETE OPERATIONS */
const notDone = operations.filter(op => !completed[op.id]);
console.log("Not done operations:", notDone.length);
notDone.filter(op => !op.dep.every(d => completed[d] || notDone.find(n => n.id === d))).forEach(op => {
    console.log(op)
    console.table(op.dep.map(d => [d, completed[d], notDone.find(n => n.id === d)]))
});
/* TRANSLATE ALL IDS */
const fin = inOrder.filter(op => !(op.triggers.length > 0 && op.triggers.length === optimized[op.id]?.length));
fin.forEach(op => {
    op.id = translate[op.oid = op.id];
    op.dep = op.dep.map(d => translate[d]);
    op.triggers = op.triggers.map(d => translate[d]);
});
/* CREATE LOGIC */
let code = "let counter2=0;\n" + fin.map(op => op.id + " = " + op.act + "; // " + op.oid).join("\n");
fs.writeFileSync(join(__dirname, "ops_sorted.js"), code);
fs.writeFileSync(join(__dirname, "ops_sorted.json"), JSON.stringify(fin, null, 2));
fs.writeFileSync(join(__dirname, "ops_not_reached.js"), JSON.stringify(notDone, null, 2));
```

This sorts the operation list in execution order, and assign a new id to all of them, indexed by execution order.

I also attempted to do some optimization but soon decided that they are unnecessary.

```json
{
    "id": "a37",
    "dep": [
      "a35",
      "a33",
      "a33",
      "a35"
    ],
    "act": "max(a35, a33)",
    "tryCollapse": true,
    "triggers": [
      "a47",
      "a50"
    ],
    "oid": "DVeHifByrMKQObal"
},
```

### Script File

The sorting process also generated a script file with all logic intact, but without the sea of fake promises.

```js
let counter2=0;
a0 = 1n; // kSnxEdOONuFSkRnE
a1 = counter2++ * 173n + BigInt((prompt() || "").charCodeAt() || 0); // xqAtWqFzGFbQMtuc
a2 = counter2++ * 173n + BigInt((prompt() || "").charCodeAt() || 0); // QSBflguWRIeDgcYv

...

a30 = counter2++ * 173n + BigInt((prompt() || "").charCodeAt() || 0); // sYhhPYfwUQnDLiqt
a31 = counter2++ * 173n + BigInt((prompt() || "").charCodeAt() || 0); // lPbEHghNFNWoAxPm
a32 = counter2++ * 173n + BigInt((prompt() || "").charCodeAt() || 0); // rKeXgPFlZdvFESNH
a33 = max(a1, a15); // szhocENPfZdnAvZb
a34 = min(a1, a15); // nHrEeeJeABGhYQpU
a35 = max(a25, a34); // vpzhAONTnucYpOKC
a36 = min(a25, a34); // PavkasYxsHLHgagA
a37 = max(a35, a33); // DVeHifByrMKQObal
...

a5149 = a5148 & BigInt(!a2945); // FmxmEEawtCtzgkjj
a5150 = a5149 & BigInt(!a3008); // CGKXThEqPwKPpZJk
a5151 = a5150 & BigInt(!a3072); // GRyibuMaolVUVMTH
a5152 = a5151; // QlIDhWbkomaROiuV
a5153 = a5151 ? alert("correct") : alert("wrong"); // last step
```

We can see that this seems to be verifying 32 characters against some fancy 5000 operation `(str) => bool` function.

## Solving it

I first tried writing my own backpropagation logic, but all the variables $a_1\dots a_{32}$ are only narrowed down to... $a_n\in[70,5488]$.

It turns out that my own code is not good at dealing with `max` `min` chains with a lot of unknown variables (It can handle everything else though).

So I decided to use Z3.

```js
import fs from "fs";
import { join, dirname } from "path";
import { fileURLToPath } from 'url';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
const ops_sorted = JSON.parse(fs.readFileSync(join(__dirname, "ops_sorted.json"), "utf8"));

const operations = {};
let code = `
from z3 import *

s = Solver()
n = -1

def min(a, b):
    return If(a <= b, a, b)
def max(a, b):
    return If(a >= b, a, b)
`;
ops_sorted.forEach(op => operations[op.id] = op);
for (const op of ops_sorted) {
    if (op.act === 'counter2++ * 173n + BigInt((prompt() || "").charCodeAt() || 0)') {
        op.act = `(n := n+1) * 173 + _${op.id}`
        code += `_${op.id} = BitVec('${op.id}', 32)\n`
        code += `s.add(_${op.id} >= 0x20, _${op.id} < 0x7e)\n`
        code += `${op.id} = ${op.act} # ${op.oid}\n`;
        continue;
    }
    if (op.act === 'a5151 ? alert("correct") : alert("wrong")') {
        code += `s.add(a5151 == True) # ${op.oid}\n`;
        continue;
    }
    op.act = op.act.replace(/(\d+)n/g, '$1')
        .replace(/BigInt/, '')
        .replace(/!/, '~')
        .replace(/ & 1$/, ' & 1 == 1')
        .replace(/^1$/, 'Bool(True)')
    code += `${op.id} = ${op.act} # ${op.oid}\n`;
}
code += `
if s.check() == sat:
    m = s.model()
    dat = [[v,m[v]] for v in m][:-1]
    sorted_dat = sorted(dat, key=lambda x: int(str(x[0])[1:]))
    flag = [chr(x[1].as_long()) for x in sorted_dat]
    print(''.join(flag))
else:
    print("no sol")
    `;
fs.writeFileSync(join(__dirname, "z3.generated.py"), code);
```

### Generated Python File

```python
from z3 import *

s = Solver()
n = -1

def min(a, b):
    return If(a <= b, a, b)
def max(a, b):
    return If(a >= b, a, b)
a0 = Bool(True) # kSnxEdOONuFSkRnE
_a1 = BitVec('a1', 32)
s.add(_a1 >= 0x20, _a1 < 0x7e)
a1 = (n := n+1) * 173 + _a1 # xqAtWqFzGFbQMtuc
_a2 = BitVec('a2', 32)
s.add(_a2 >= 0x20, _a2 < 0x7e)
a2 = (n := n+1) * 173 + _a2 # QSBflguWRIeDgcYv
_a3 = BitVec('a3', 32)

...

a5150 = a5149 & (~a3008) # CGKXThEqPwKPpZJk
a5151 = a5150 & (~a3072) # GRyibuMaolVUVMTH
a5152 = a5151 # QlIDhWbkomaROiuV
s.add(a5151 == True) # last step

if s.check() == sat:
    m = s.model()
    dat = [[v,m[v]] for v in m][:-1]
    sorted_dat = sorted(dat, key=lambda x: int(str(x[0])[1:]))
    flag = [chr(x[1].as_long()) for x in sorted_dat]
    print(''.join(flag))
else:
    print("no sol")
```

```flag
FLAG{pr0M1S3s_@ND_a5YnC'n_@w@17}
```

And there we go.

## Extra: Backpropagation

Spent a lot of time on this, a script that starts from the end, and propagates up, solving for values as it goes. Didn't work sadly.

- Find all operations whose dependencies have all been visited
- Visit the operation and finalize its value.
- Parse the operation's logic, and determine what values its parents should have (`triggers`).
- Add those prospective values to `pending_vals`

```js
import fs from "fs";
import { join, dirname } from "path";
import { fileURLToPath } from 'url';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
const ops_sorted = JSON.parse(fs.readFileSync(join(__dirname, "ops_sorted.json"), "utf8"));

const pending_val_type = {};
const pending_vals = {};
const operations = {};
const visited = {};
const values = {};
ops_sorted.forEach(op => operations[op.id] = op);

function computeBounds(vals) {
    let min = -Infinity, max = Infinity;
    for (const [op, v] of vals) {
        if (op === "max") max = Math.min(max, v);
        else if (op === "min") min = Math.max(min, v);
        else throw new Error(`Unknown minmax operation: ${op}`);
    }
    return [min, max];
}

function addPendingVal(id, val, type = "identity") {
    if (val === undefined || val === null || val === NaN) throw new Error(`Invalid value for ${id}: ${val}`);

    pending_vals[id] ??= [];
    // downgrade type if needed
    if (typeof val !== "number") {
        if (val[0] === 'minmax') {
            pending_val_type[id] = type = 'minmax';
            const [_, min, max] = val;
            if (min > max) throw new Error(`Invalid minmax value for ${id}: ${val}`);
            pending_vals[id].push(["min", min])
            pending_vals[id].push(["max", max]);
            return;
        }

        if (type === 'identity') throw new Error(`Non-number value for ${id}: ${val} (${val.constructor.name})`);
    }
    // upgrage type if needed
    if (pending_val_type[id] && pending_val_type[id] !== type) {
        // throw new Error(`Pending value type mismatch for ${id}: ${pending_val_type[id]} !== ${type}`);
        if (pending_val_type[id] === "identity") return; // ignore if identity

        if (type !== "identity") throw new Error(`Pending value type mismatch for ${id}: ${pending_val_type[id]} !== ${type}, new type not identity`);
        switch (pending_val_type[id]) {
            case "minmax":
                const [min, max] = computeBounds(pending_vals[id]);
                if (!(min <= val && val <= max)) throw new Error(`New identity value ${val} not in range ${min} - ${max} for ${id} ${JSON.stringify(pending_vals[id])}`);
                pending_vals[id] = [];
                break;
            case "bitwise":
                const uniqueNs = new Set(pending_vals[id].map(([n, v]) => n));
                if (uniqueNs.size !== pending_vals[id].length)
                    throw new Error(`Duplicate n values found in bitwise pending_vals[${id}]: ${JSON.stringify(pending_vals[id])}`);
                const oldV = pending_vals[id].reduce((acc, [n, v]) => acc | (v << n), 0x0);
                if ((oldV & val) !== oldV) throw new Error(`${id} New identity value ${val}!=${oldV} ${JSON.stringify(pending_vals[id])}`);
                pending_vals[id] = [];
                break;
            default:
                throw new Error(`Unknown pending value type for ${id}: ${pending_val_type[id]}`);
        }
    }
    pending_val_type[id] = type;
    pending_vals[id].push(val);
}
const unsureValues = {};
function finalizePendingVal(id) {
    const op = operations[id];
    if (!pending_vals[id]) return undefined;
    if (op.triggers.length > 0 && !op.triggers.every(d => visited[d])) throw new Error(`Not all child visited for ${id} ${JSON.stringify(Object.fromEntries(op.dep.map(d => [d, !!visited[d]])))}`);
    switch (pending_val_type[id]) {
        case "identity":
            if (!pending_vals[id].every(v => v === pending_vals[id][0])) throw new Error(`Identity operation with different values: ${JSON.stringify(pending_vals[id])}`);
            return pending_vals[id][0];
        case "minmax":
            const [min, max] = computeBounds(pending_vals[id]);
            if (min === -Infinity || max === Infinity) throw new Error(`Min/max operation with missing values: ${JSON.stringify(pending_vals[id])}`);
            if (min === max) return min;
            unsureValues[id] = {
                id,
                range: [min, max],
                triggers: op.triggers,
                peers: op.triggers.map(d => operations[d].dep).flat()
            };
            return ['minmax', min, max];
        // else throw new Error(`Min/max operation with different values: ${min} / ${max} | ${JSON.stringify(pending_vals[id])}`);
        case "bitwise":
            const uniqueNs = new Set(pending_vals[id].map(([n, v]) => n));
            if (uniqueNs.size !== pending_vals[id].length)
                throw new Error(`Duplicate n values found in bitwise pending_vals[${id}]: ${JSON.stringify(pending_vals[id])}`);
            return pending_vals[id].reduce((acc, [n, v]) => acc | (v << n), 0x0);
        default:
            throw new Error(`Unknown pending value type for ${id}: ${pending_val_type[id]}`);
    }
}
function shouldVisit(id) {
    if (visited[id]) return false;
    const op = operations[id];
    return op.triggers.length === 0 || op.triggers.every(d => visited[d]); // triggers = operations that depend on self
}

function visit(id) {
    if (visited[id]) throw new Error(`Already visited ${id}`);
    visited[id] = true;
    const op = operations[id];
    let val = values[id] = finalizePendingVal(id);
    // console.log(`finalized ${id} = ${val}`);
    if (val === undefined) return false;

    const act = op.act.replace(/ \? alert\("correct"\) : alert\("wrong"\)/, "");
    let matches;
    if (matches = act.match(/^(a\d+)$/)) { // IDENTITY
        const [, a] = matches;
        addPendingVal(a, val);
    } else if (matches = act.match(/^(\d+)n?$/)) { // CONSTANT
        const [_, nval] = matches;
        if (val !== parseInt(nval)) throw new Error(`${id}: ${val} !== ${nval}!`);
    } else if (matches = act.match(/^(a\d+) & 1n$/)) { // AND 1
        const [, a] = matches;
        if (val !== 0 && val !== 1) throw new Error(`${id}: AND operation with non binary ${val}!`);
        addPendingVal(a, [0, val], 'bitwise');
    } else if (matches = act.match(/^(a\d+) & BigInt\((!?)(a\d+)\)$/)) { // a AND b = 1
        const [, a, f, b] = matches;
        if (val === 1) {
            addPendingVal(a, 1);
            addPendingVal(b, f ? 0 : 1);
        } else
            throw new Error(`${id}: 'a AND b' operation with non-1 ${val}!`);
    } else if (matches = act.match(/^(min|max)\((a\d+), (a\d+)\)$/)) { // MIN/MAX(a,b) = val
        const [, op, a, b] = matches;
        if (typeof val !== "number" && val[0] === 'minmax') {
            val = op === "min" ? val[1] : val[2];
        }
        addPendingVal(a, [op, val], 'minmax');
        addPendingVal(b, [op, val], 'minmax');
    } else if (matches = act.match(/^(a\d+) >> (\d+)n$/)) { // a >> n
        const [, a, n] = matches;
        if (val !== 0 && val !== 1) throw new Error(`${id}: Bitwise operation with non binary ${val}!`);
        addPendingVal(a, [parseInt(n), val], 'bitwise');
    } else if (act === 'counter2++ * 173n + BigInt((prompt() || "").charCodeAt() || 0)') {
        console.log(`found ${id}: ${val}`);
    } else {
        throw new Error(`${id}: Unexpected operation '${act}' = ${val}!`);
    }
}
addPendingVal("a5153", 1);
addPendingVal("a5152", 1);

function start() {
    while (Object.keys(visited).length < Object.keys(operations).length) {
        const visiting = ops_sorted.filter(op => shouldVisit(op.id));
        // console.log("Visiting with", visiting.length, "operations");
        visiting.forEach(op => visit(op.id));
    }
}

start();
```
