---
ai_date: '2025-04-27 05:24:13'
ai_summary: Flag obtained by finding numbers with hash starting '000000', exploiting
  repeated number acceptance
ai_tags:
- hash
- pattern-attack
- repeated-numbers
created: 2024-06-21T19:41
points: 143
solves: 250
updated: 2024-08-05T09:53
---

> compute hash to get your flag
> ハッシュを計算してフラグを取ろう

The code seems to be finding numbers whose hash start with `000000`.

With some modification to the code, we can see the numbers.

```
progress: 0 / 1000000 0
2862152
progress: 1 / 1000000 1
7844289
```

Manually poking the server with these numbers made me realize that the server is happy with repeated numbers.

```js
for(let i=0;i<10;i++)await send(new Array(100000).fill("2862152"));
```

Run the above in the js console and we will get the flag.

```flag
FLAG{N0nCE_reusE_i$_FUn}
```