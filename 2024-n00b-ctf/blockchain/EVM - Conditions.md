---
ai_date: '2025-04-27 05:19:31'
ai_summary: Recovering the flag from decompiled EVM bytecode arithmetic check
ai_tags:
- decomp
- arithm
- exploit
created: 2024-08-04T06:14
points: 463
solves: 154
tags:
- web3
updated: 2024-08-05T19:37
---

Just decompile the EVM byte code. [app.dedaub.com/decompile](https://app.dedaub.com/decompile)

`5f600f607002610258525f60056096046090525f600760090A61FFFA526105396126aa18620bfabf52600361fffa5102620bfabf51013461025851600402016090510114604857ff00`

```js
function function_selector() public payable { 
    assert(6750 + msg.value != 0xdb15fe);
    selfdestruct(0);
}
// 0xdb15fe-6750 = dafba0
```

```flag
n00bz{0xdafba0}
```