---
ai_date: 2025-04-27 05:22:16
ai_summary: Decoding function revealed JSFuck-encoded text, leading to flag extraction after decoding.
ai_tags:
  - jsfuck
  - decode
  - rev
created: 2024-09-07T18:46
points: 400
solves: 14
updated: 2025-07-14T09:46
---

After [deobfuscation](https://obf-io.deobfuscate.io/) and slight modifications of the original code, I discovered a simple decoding function that is used everywhere, so I ported it over to node and dumped all decoded strings into one file.

```js [code.js]
// this is the decoder used for all the functions
x._d = str => {
    str = decodeURI(str);
    let b64 = '';
    for (let i = 0; i < str.length; i++) {
        b64 += x.pgd[600](str.charCodeAt(i) ^ x.kill.charCodeAt(i % x.kill.length));
    }
    console.log(atob(b64))
    return atob(b64);
};
```

```js [code.js]
... // ported to node js
const window = new JSDOM().window;
const document = window.document;
global.x = window;
global.z = document;
global.d = document;
x.d = x.z;
d.ce = d.createElement
window.arts = [];
main(119);
main(533);
main(778);
main(309);
main(353);
main(647)();
```

```sh
node code.js > out.txt
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1725749379/2024/09/3d0630c8bdcf49b83a5dfc72fcb453d6.png)

There appears to be a huge chunk of text.

A snippet looks like this `[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!`.

This appears to be the result of `jsfuck`, (how many times have we met?)

Running it directly didn't work, let's try decoding it.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1725749673/2024/09/098b280b947cdc86c3eb6b8456ad9e99.png)

Oh.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1725749684/2024/09/4d84df35c7f75a3e628d47fe1e87e1ff.png)

```flag
UCTF(RIG-E-JENN)
```

And we have our flag.

This should not be in `web`, it should be in `rev` or `misc`