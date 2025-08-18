---
ai_date: 2025-08-17 20:46:32
ai_summary:
  Exploited prototype chain to call functions in WebAssembly, limited to
  numerical arguments due to memory pointers.
ai_tags:
  - wasm
  - prototype
  - function-calling
created: 2025-08-16T07:16
tags:
  - unsolved
title: Captivating Canvas Contraption
updated: 2025-08-17T20:46
---

After some trial and error (namely making the editor run the wasm on the spot for faster testing), I found that I can actually read and call functions in `__proto__`.

```js
// run this in the console first to populate prototype
({}).__proto__.flag = (function () {
  console.log("flag called", [...arguments]);
})({}).__proto__.secret = 2;
```

```ts
@external("__proto__", "secret")
declare const sec: i32;

@external("__proto__", "flag")
declare function flag(code:string): void;

export function renderPixel(x: i32, y: i32, dt: f32): i32 {
    flag("yoo");
    return sec;
}
```

However the problem with this is that all external functions are called with arguments being pointers to memory locations within the wasm.

```
flag called
Array [ 1056 ]
renderPixel(1, 2, 3) 2
```

And it seems that this is a limitation of web assembly itself.

It means that we can only use numerical arguments, and some function on prototype chain is callable.

I got stuck here.
