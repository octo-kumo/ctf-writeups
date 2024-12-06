---
created: 2024-11-30T17:09
updated: 2024-11-30T17:09
---

I hope everyone liked my drawing!

## Step 1
- https://obf-io.deobfuscate.io/
- Remove dead code

You will realize that only 1 function is relevant.

```js
function check(_0x266792) {
    ...
}
```

## Step 2: Operators
- Undo the base64

You will realize that all of these base64 strings are actually operators done on a stack

The meaning of `x` and `y` may be obvious or may be not, but that's ok, we will move on to the next part.

```js
function check(flag) {
    const arr1 = [3, 0, 4, 0, 5, '0', '3', '0', ...];
    const arr2 = ["CzhQSPrjvxQ7vfEm", "spCiy27WpEuz0bAh", ...];
    const operators = {
        CzhQSPrjvxQ7vfEm: 'function (x, y) {\r\n' +
            '        let a = y[x.pop()], b = y[x.pop()]\r\n' +
            '        return b[a]\r\n' +
            '    }',
    ...
    };
    const arr3 = ["d3dme2Zha2VfZmxhZ30=", "d3dme2Zha2VfZmxhZ30=", "d3dme2Zha2VfZmxhZ30=", 17, 19, "length", ...];
    const arr4 = [];
    for (let i = 0; i < flag.length; i++) {
        arr3[i] = flag[i];
    }
    function func(i) {
        arr3.push(i);
        return arr3.length - 1;
    }
    arr1.forEach(el => {
        if (typeof el === "string") {
            const op = eval('(' + operators[arr2[parseInt(el)]] + ')');
            arr4.push(func(op(arr4, arr3)));
        } else {
            arr4.push(el);
        }
    });
    return arr3[arr4.pop()];
}
```

> At this point you could avoid everything else by writing your own hook and run the code above to see what operations are being made, and what value is being returned, after which you could solve for the flag.

## Step 3: Understand the arrays
What are the meanings of the arrays?

```js
function func(i) {
    arr3.push(i);
    return arr3.length - 1;
}
arr1.forEach(el => {
    if (typeof el === "string") {
        const op = eval('(' + operators[arr2[parseInt(el)]] + ')');
        arr4.push(func(op(arr4, arr3)));
    } else {
        arr4.push(el);
    }
});
```

Over here you may notice that the operators in the previous step are used with `arr4` and `arr3` as input.
New data produced are added to `arr3`, and the index is pushed to `arr4`.
Furthermore, in the operators, `arr3` is being accessed via indexers from `arr4`.
We can deduce that `arr4` stores indexes to values in `arr3`.

`arr2` on the other hand seems to just store all the keys of the operators, which means `parseInt(el)` will result in an operator, this means `arr1` is most likely an operator stack. However if `el` is not a string int, it seems to store values used in `arr4` which means `arr1` stores both operators and `arr3` indexes.

- All actual data is in `arr3`, let's call it `mem`.
- `arr2` is just operators, it can be called `ops`.
- `arr1` is operator indexes + memory indexes, it can be called `prog`.
- `arr4` is used purely as a stack, it can be called `stack`.

```js
function check(flag) {
    const prog = [3, 0, 4, 0, 5, '0', '3', ...];
    const ops = ["CzhQSPrjvxQ7vfEm", "spCiy27WpEuz0bAh", ...];
    const operators = {
        CzhQSPrjvxQ7vfEm: 'function (x, y) {\r\n' +
            '        let a = y[x.pop()], b = y[x.pop()]\r\n' +
            '        return b[a]\r\n' +
            '    }',
        ...
    }
    const mem = ["d3dme2Zha2VfZmxhZ30=", "d3dme2Zha2VfZmxhZ30=", "d3dme2Zha2VfZmxhZ30=", 17, 19, ...];
    const stack = [];
    for (let i = 0; i < flag.length; i++) {
        mem[i] = flag[i];
    }
    function store_in_mem(i) {
        mem.push(i);
        return mem.length - 1;
    }
    prog.forEach(el => {
        if (typeof el === "string") {
            const op = eval('(' + operators[ops[parseInt(el)]] + ')');
            stack.push(store_in_mem(op(stack, mem)));
        } else {
            stack.push(el);
        }
    });
    return mem[stack.pop()];
}
```

> You may or may not recognize this as a [RPN (Reverse Polish notation)](https://en.wikipedia.org/wiki/Reverse_Polish_notation) evaluator.

## Step 4: Decoding the program

Now we need to actually decode what `prog` stores.

We can use hooks, or simply make all the operators return strings.

Two of the functions may appear complicated but if you test them, they are just addition and subtraction.

```js
const operators = {
    CzhQSPrjvxQ7vfEm: 'function (x, y) {\r\n' +
        '        let a = y[x.pop()], b = y[x.pop()]\r\n' +
        '        return `${b}[${a}]`\r\n' +
        '    }',
    spCiy27WpEuz0bAh: 'function (x, y) {\r\n        return `${y[x.pop()]}.charCodeAt(0)`\r\n    }',
    mTvc3QBx6ieTIzEA: 'function (x, y) {\r\n        return `!${y[x.pop()]}`\r\n    }',
    I4TO8mHsfL6Tic7v: 'function (x, y) {\r\n' +
        '        let a = y[x.pop()], b = y[x.pop()]\r\n' +
        '        return `${b} % ${a}`\r\n' +
        '    }',
        ...
};
```

Now run it but with the arguments wrapped.

```js
console.log(check(["'wwf{flag}'", '[155, 25, 81, 18, 37, 247, 169, 26]', '[239, 17, 117, 197, 235, 182, 242, 83]']))
```

We now get the actual program!

> remember to add brackets for correct execution priority, I did not add brackets.

```js
4991038 === 194 * 198 + [239, 17, 117, 197, 235, 182, 242, 83][18 % [239, 17, 117, 197, 235, 182, 242, 83][length]] * [155, 25, 81, 18, 37, 247, 169, 26][18 % [155, 25, 81, 18, 37, 247, 169, 26][length]] ^ 'wwf{flag}'[36 % 'wwf{flag}'[length]].charCodeAt(0) + 'wwf{flag}'[37 % 'wwf{flag}'[length]].charCodeAt(0) ^ [239, 17, 117, 197, 235, 182, 242, 83][18 % [239, 17, 117, 197, 235, 182, 242, 83][length]] + ...
```

## Step 5: Simplification of the program / Direct solve

```js
const operators = {
    CzhQSPrjvxQ7vfEm: function (x, y) {
        let a = y[x.pop()], b = y[x.pop()]
        if (Array.isArray(b)) {
            if (!b[a]) console.log(b, a)
            return b[a]
        }
        if (a === 'length') return `${b}.length`
        return `${b}[${a}]`
    },
    spCiy27WpEuz0bAh: function (x, y) {
        return `${y[x.pop()]}`
    },
    mTvc3QBx6ieTIzEA: function (x, y) { return `!${y[x.pop()]}` },
    I4TO8mHsfL6Tic7v: function (x, y) {
        let a = y[x.pop()], b = y[x.pop()]
        if (typeof a === 'number' && typeof b === 'number') return b % a;
        return `(${b} % ${a})`
    },
    UpIk0FsWtwynGyBU: function (x, y) {
        let a = y[x.pop()], b = y[x.pop()]
        if (typeof a === 'number' && typeof b === 'number') return a * b;
        return `(${a} * ${b})`
    },
    HGP5hbB7yJzI2iuN: function (x, y) {
        let a = y[x.pop()], b = y[x.pop()]
        if (typeof a === 'number' && typeof b === 'number') return 1 / (a / b);
        return `(${b} / ${a})`
    },
    '8ZuAtV6T1A4EaCzU': function (x, y) {
        let a = y[x.pop()], b = y[x.pop()]
        if (typeof a === 'number' && typeof b === 'number') return a + b;
        return `(${a}+${b})`
    },
    GkzKiFsahtTuIhWZ: function (x, y) {
        let b = y[x.pop()], a = y[x.pop()]
        if (typeof a === 'number' && typeof b === 'number') return a - b;
        return `(${a} - ${b})`
    },
    iGXbPUsu9ti82rZ3: function (x, y) { return `(${y[x.pop()]} === ${y[x.pop()]})` },
    Ajsp9ey55YxDO6Dh: function (x, y) { return `(${y[x.pop()]} !== ${y[x.pop()]})` },
    CTI3do19ytT13s0V: function (x, y) { return `(${y[x.pop()]} & ${y[x.pop()]})` },
    Iv1du7HLwfTBhC33: function (x, y) { return `(${y[x.pop()]} ^ ${y[x.pop()]})` },
    FVGaT0YAtvEnrh1L: function (x, y) { return `(${y[x.pop()]} | ${y[x.pop()]})` },
    mDA2bNR6EFYLd7Zp: function (x, y) { return `(${y[x.pop()]} && ${y[x.pop()]})` },
    tVd8iQXoWejgiKNZ: function (x, y) { return `(${y[x.pop()]} || ${y[x.pop()]})` }
}
```

The program will look like this after substitution of value and reformatting.

```js
((4991038 === (194 * ((198 + (117 * (81 ^ (flag[(36 % flag.length)] +
    flag[(37 % flag.length)])))) ^ ((117 + (81 * (
    flag[(38 % flag.length)] ^ flag[(37 %
        flag.length)]))) ^ 123)))) && ((342408 === (88 * ((80 + (
    17 * (25 ^ (flag[(35 % flag.length)] +
        flag[(36 % flag.length)])))) ^ ((17 + (
    25 * (flag[(37 % flag.length)] ^ flag[
        (36 % flag.length)]))) ^ 16)))) && ((59685 === (1 * ((
    230 + (239 * (155 ^ (flag[(34 % flag
        .length)] + flag[(35 %
        flag.length)])))) ^ ((239 + (155 * (
    flag[(36 % flag.length)] ^
    flag[(35 % flag.length)]))) ^ 193)))) && ((452012 ===
    (44 * ((74 + (83 * (26 ^ (flag[(33 % flag
        .length)] + flag[(34 %
            flag.length)])))) ^ ((83 + (26 * (
        flag[(35 % flag.length)] ^
        flag[(34 % flag.length)]))) ^ 171)))) && ...
```

Plugging it into z3 will give you the flag.

```flag
wwf{m45h1r0_w41fu_>_<_50_cu73~~_4hw4_}
```
