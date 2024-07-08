---
created: 2024-06-11T01:17
updated: 2024-07-07T23:08
---

```js
"use strict";
var a = '1';
var b = '2';
var c = '3';
var d = '4';
var e = '5';
var f = '&ŧĳŧŠ#ŇŠŇĳ,ŞŨųŞ)ňŲŞų2ŮįŞń?ŀŒŘŞ8őİŦŧ\x05ųľ';
var h = '';
for (var i = 0; i < f.length; i++) {
    if (i % 5) {
        h = f[i];
        h = String.fromCharCode(h.charCodeAt() - a.charCodeAt());
        h = String.fromCharCode(h.charCodeAt() - b.charCodeAt());
        h = String.fromCharCode(h.charCodeAt() - c.charCodeAt());
        h = String.fromCharCode(h.charCodeAt() - d.charCodeAt());
        h = String.fromCharCode(h.charCodeAt() - e.charCodeAt());
        g += h
    } else {
        h = i ^ 38;
        h = String.fromCharCode(h);
        if (f[i] != h) {
            alert("SomeThing Wrong,,");
            break
        }
    }
}
var flag = g;
```

It seems that the only thing missing is variable declaration of `g`

Adding `var g = '';` fixes the problem

![](https://res.cloudinary.com/kumonochisanaka/image/upload/v1718083520/2024/06/c6b6b9f73806b24be67d7ff72326a48c.png)

Indeed it is

```text
CDDC22{h4haHaH4_it_Is_to0_EASY_R1ght?}
```
