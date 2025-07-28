---
created: 2025-07-28T02:17
updated: 2025-07-28T06:35
description: Was it too hard?
solves: 0
points: 500
---

> World Wide Flags 2
>
> 500
>
> `author: yun`
>
> Last year's WWF challenge was too easy~ We made it impossible this time <3
>
> You have to guess 4 flags at the same time, 100 times. Good luck!
>
> [https://flags2.chall.wwctf.com](https://flags2.chall.wwctf.com)
>
> hint 1
> i made a writeup on last year's challenge
>
> hint 2
> secret.wasm and secret.js seems very sus, what even is `get_cid`?
>
> hint 3
> if you think about it, wwf 2 is web, crypto, osint, rev, misc all in one.

## flag parsing
The `osint` part (and `ai`).

https://yun.ng/c/ctf/2024-wwctf/misc/world-wide-flags

Just copy this (and remove some unneeded steps).

## analysis
The `rev` part.

Now we need to figure out what the server is doing, because at lv 69 we get pure noise.

```js
import init, { get_key, get_cid, get_flags } from './secret';
// ...
const ENC_ALGO = 'aes-256-gcm';
const ENC_SEC = crypto.randomBytes(32);
// ...
export function get_jwt(KEY, solved, progress, sub) {
    if (!KEY) {
        KEY = get_key();
        solved = [];
        progress = 0;
        sub = crypto.randomUUID();
    }

    if (progress >= TOTAL_TO_SOLVE) return {
        progress,
        solved: true
    }

    let [cid, salt] = get_cid(KEY);
    while (solved.includes(cid)) [cid, salt] = get_cid(KEY);
    const flags = get_flags(KEY, progress);
    const nonce = crypto.randomBytes(16);
    const iv = crypto.randomBytes(12);
    const h = crypto.createHash('sha256');
    for (const fi of flags) {
        h.update(fi.toString() + ";");
    }
    h.update(cid + ';');
    h.update(salt + ';')
    h.update(nonce);
    const checksum = h.digest('base64');
    const jti = crypto.randomUUID();
    const exp = Math.floor(Date.now() / 1000) + JWT_EXP;
    FLAGS[jti] = flags;

    const secret = crypto.createCipheriv(ENC_ALGO, ENC_SEC, iv);
    const info = JSON.stringify({
        key: KEY,
        solved: Array.from(solved),
        cid,
        salt,
    });
    const enc = Buffer.concat([secret.update(info), secret.final()]);

    const payload = {
        sub,
        progress,
        iv: iv.toString('base64'),
        nonce: nonce.toString('base64'),
        enc: enc.toString('base64'),
        checksum: checksum,
        jti,
        exp,
        tag: secret.getAuthTag().toString('base64'),
        imp: progress >= IMPOSSIBLE ? 1 : 0,
    };
    return payload
}
```

This is how the flags are selected.

We have access to:
- `progress`
- `iv` not useful
- `nonce` used for checksum
- `enc` we can't decrypt this, because the secret key is only on the server, and aes isn't really the best thing to crack.
- `checksum` `sha256('f1;f2;f3;f4;cid;salt;nonce')` but we don't know `cid` and `salt` so we can't actually brute force crack, because it is just too many `sha256` to crack.
- `tag` for `enc`

We can't really figure out anything other than that `cid` and `salt` could be useful, however even if we know them we can't brute force the flags, $234^4=2,998,219,536$.

Let's analyse the `./secret` library.
We can test it locally and see what the functions do.

```js
import init, { get_key, get_cid, get_flags } from "./secret";
await init();
const key = get_key();
console.log(get_key());
// 4275175899065959709199306424807125952282106145263950575176514585345191378053505748312644688911950927136846183704084923799477446092404041673156492448732779
console.log(get_cid(key));
// [ "223", "52" ]
console.log(get_cid(key));
// [ "607", "202" ]
console.log(get_cid(key));
// [ "881", "839" ]
console.log(get_flags(key, 1));
// Uint32Array(4) [ 145, 53, 78, 76 ]
console.log(get_flags(key, 1));
// Uint32Array(4) [ 145, 53, 78, 76 ]
console.log(get_flags(key, 2));
// Uint32Array(4) [ 110, 231, 113, 110 ]
```

- `get_key` appears to return a large prime number
- `get_cid` appears to return 2 numbers, the first is a prime, the second we don't know.
- `get_flags` seems to just return the flag id for any given level and appears to be deterministic.

It would be clear that to pass the challenge beyond lv70, we need to crack the key, then replicate `get_flags` or just call it.

So how?

We need to figure out the `wasm` contains, because `secret.js` is just a loader.

```bash
$ wasm-decompile secret.wasm > out.w
```

### `get_cid`

The `crypto` part.

```
// line 7225 - 7717
export function get_cid(a:{ a:int, b:int }, b:int, c:int_ptr) { // func46
...
}
```

You might realise purely by crypto instinct, by playing around or by actually reading the wasm, that the two numbers `a` `b` return by the `get_cid` function has the following properties.

$$
K\bmod a=b
$$

Where `K` is the key, `a` is the first random prime returned by `get_cid` and `b` is the residue.

> You either need to have a crypto player or a godly rev player for this lol.

Anyways now that you know `get_cid` leaks some information about the key.

But you know neither `cid` nor `salt`, so how is that helpful?

### bruteforce

Remember how `checksum = sha256('f1;f2;f3;f4;cid;salt;nonce')`?

And how `cid` and `salt` are both very small numbers?

```
cid distribution
---
min: 3
max: 1021
     417.00 ┼╮                                                   
     380.80 ┤│                                                   
     344.60 ┤│                                                   
     308.40 ┤│╭╮     ╭╮                                          
     272.20 ┤││╰╮╭╮  ││╭╮                 ╭╮╭╮                   
     236.00 ┤││ ││╰╮╭╯│││╭╮  ╭╮  ╭╮╭╮     ││││       ╭╮          
     199.80 ┤╰╯ ││ ││ │││││╭╮││  ││││   ╭╮││││       ││          
     163.60 ┤   ╰╯ ╰╯ ││╰╯╰╯││╰──╯││╰╮ ╭╯╰╯││╰╮╭──╮  │╰╮╭─╮╭╮╭─╮ 
     127.40 ┤         ││    ││    ╰╯ │ │   ││ ╰╯  ╰╮ │ ││ ││╰╯ │ 
      91.20 ┤         ││    ╰╯       ╰─╯   ││      ╰─╯ ╰╯ ╰╯   │ 
      55.00 ┤         ╰╯                   ╰╯                  ╰
```

We can brute force the values of `cid` and `salt` after solving the flags.

Since we now have the values of `f1-f4` and we have `nonce`.

Ok so we can figure out `cid` and `salt` now, how do we find the key?

### the math

$$
\begin{align}
K & \equiv a_{1}\pmod{b_{1}} \\
K & \equiv a_{2}\pmod{b_{2}} \\
 & \vdots \\
K & \equiv a_{n}\pmod{b_{n}}
\end{align}
$$

We can find $K$ via chinese remainder theorem.

$$
\begin{align}
\exists M & =a_{1}\times a_{2}\times a_{3}\times\dots a_{n} \\
K & \equiv R \pmod{M} \\
R & \in [0, M) \\
K & =R+kM,k\in \mathbb{Z}
\end{align}
$$

How does this help?

Well if $M$ is large enough, it will be larger than $K$ itself, and $R=K$.

This means that with enough pairs of $(a_{cid},b_{salt})$, we can find the key.

## solve script
This script assumes that you have reversed the other parts, like the flag generation etc, but that is not necessary, you can implement this script in js instead, or call the wasm methods from python.

```python
import base64
from functools import reduce
from itertools import product
import operator
import os
import time
import cv2
import requests
import numpy as np
import jwt
import hashlib
from Crypto.Cipher import ChaCha20


def extended_gcd(a: int, b: int):
    if b == 0:
        return a, 1, 0
    g, x1, y1 = extended_gcd(b, a % b)
    return g, y1, x1 - (a // b) * y1


def modinv(a: int, m: int) -> int:
    g, x, _ = extended_gcd(a, m)
    if g != 1:
        raise ValueError(f"No inverse for {a} mod {m}, gcd={g}")
    return x % m


def crt_solve(mods: list[int], rems: list[int]) -> int:
    if len(mods) != len(rems):
        raise ValueError("mods and rems must be same length")
    M = reduce(operator.mul, mods, 1)
    x = 0
    for m_i, r_i in zip(mods, rems):
        M_i = M // m_i
        inv = modinv(M_i, m_i)
        x += r_i * M_i * inv
    return x % M


def preprocess_image(img, level=0):
    img = cv2.fastNlMeansDenoisingColored(img, None, 10, 10, 7, 21)
    if level < 20:
        img = img
    if level < 30:
        img = cv2.GaussianBlur(img, (5, 5), 0)
        img = (np.clip(img, 80, 255)-80)*255.0/(255-80)
        img = np.clip(np.round(img), 0, 255).astype(np.uint8)
    else:
        img = (np.clip(img, 128, 255)-128)*2
    return img


def get_corners(contour):
    approx = cv2.approxPolyDP(contour, 0.04 * cv2.arcLength(contour, True), True)
    if len(approx) != 4:
        return None
    corners = np.array(approx.reshape(4, 2), dtype="float32")
    s = np.sum(corners, axis=1)
    diff = np.diff(corners, axis=1)
    return corners[[np.argmin(s), np.argmin(diff), np.argmax(s), np.argmax(diff)]]


def untransform(img, rect, width=250, height=175):
    dst = np.array([
        [0, 0],
        [width - 1, 0],
        [width - 1, height - 1],
        [0, height - 1]
    ], dtype="float32")
    return cv2.warpPerspective(img, cv2.getPerspectiveTransform(rect, dst), (width, height))


def image_similarity(img1, img2):
    return cv2.matchTemplate(img1, img2, cv2.TM_CCOEFF_NORMED)[0][0].item()


WIDTH = 250
HEIGHT = 175
flags_dir = '../src/challenge/flags'
flags = sorted([f for f in os.listdir(flags_dir) if f.endswith('.png')])
flag_imgs = [cv2.resize(cv2.imread(os.path.join(flags_dir, f)), (WIDTH, HEIGHT)) for f in flags]


def solve(warped):
    scores = [(flags[i], image_similarity(warped, flag_imgs[i])) for i in range(len(flags))]
    scores.sort(key=lambda x: x[1], reverse=True)
    return scores


def full_chain(buffer, level=0, idx=0):
    img = cv2.imdecode(np.frombuffer(buffer, np.uint8), cv2.IMREAD_COLOR)
    if level >= 20:
        img = preprocess_image(img, level)
    img = cv2.resize(img, (WIDTH, HEIGHT))
    return solve(img)


primes = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97, 101, 103, 107, 109, 113, 127, 131, 137, 139, 149, 151, 157, 163, 167, 173, 179, 181, 191, 193, 197, 199, 211, 223, 227, 229, 233, 239, 241, 251, 257, 263, 269, 271, 277, 281, 283, 293, 307, 311, 313, 317, 331, 337, 347, 349, 353, 359, 367, 373, 379, 383, 389, 397, 401, 409, 419, 421, 431, 433, 439, 443, 449, 457,
          461, 463, 467, 479, 487, 491, 499, 503, 509, 521, 523, 541, 547, 557, 563, 569, 571, 577, 587, 593, 599, 601, 607, 613, 617, 619, 631, 641, 643, 647, 653, 659, 661, 673, 677, 683, 691, 701, 709, 719, 727, 733, 739, 743, 751, 757, 761, 769, 773, 787, 797, 809, 811, 821, 823, 827, 829, 839, 853, 857, 859, 863, 877, 881, 883, 887, 907, 911, 919, 929, 937, 941, 947, 953, 967, 971, 977, 983, 991, 997, 1009, 1013, 1019, 1021]


def crack_p_m(idxs, checksum, nonce):
    H = hashlib.sha256()
    for i in idxs:
        H.update(str(i).encode()+b';')
    for p in primes:
        h = H.copy()
        h.update(str(p).encode()+b';')
        for i in range(1, p):
            hh = h.copy()
            hh.update(str(i).encode()+b';')
            hh.update(nonce)
            if hh.digest() == checksum:
                return p, i
    return None, None


start = time.time()
s = requests.Session()
solved = 0
total = 100
crt_mods = []
crt_rems = []

solved_key = None
_last_rec = 0


def random_flag(key: int, i: int) -> int:
    key_truncated = key & ((1 << (8 * 32)) - 1)
    seed_bytes = key_truncated.to_bytes(32, 'big')
    cipher = ChaCha20.new(key=seed_bytes, nonce=b'\x00' * 8)
    for _ in range(i + 2):
        block = cipher.encrypt(b'\x00' * 4)
        val = int.from_bytes(block, 'little')
    return val % 234


target = "http://localhost:1337"
target = "https://flags2.chall.wwctf.com"
no_find_counter = 0
while solved < total:
    imgs = [s.get(f'{target}/flag/{i}').content for i in range(4)]

    while any([img is None for img in imgs]):
        imgs = [s.get(f'{target}/flag/{i}').content if img is None else img for i, img in enumerate(imgs)]
        time.sleep(0.5)
    token = s.cookies.get('token')
    token = jwt.decode(token, options={"verify_signature": False})

    if solved_key is not None:
        anss = [random_flag(solved_key, solved*4+i) for i in range(4)]
        anss = [flags[i][:2] for i in anss]
        print(f"{anss}, score={score}")
        res = s.post(f'{target}/solve', data={'ans[]': anss})
        res = res.json()
        print(res)
        if res['message'] == 'Correct!':
            solved = res['solves']
            continue
    scores = [full_chain(img, solved, i) for i, img in enumerate(imgs)]
    TOP_K = 3
    top3 = [slot[:TOP_K] for slot in scores]

    choice_width = max(len(str(choice)) for slot in top3 for choice, _ in slot)
    conf_width = 7
    for idx, slot in enumerate(top3):
        parts = [
            f"{str(choice):<{choice_width}} {conf:>{conf_width}.4f}"
            for choice, conf in slot
        ]
        print(f"flag {idx}: " + " | ".join(parts))

    combs = [(
        top3[0][i][0][:2],
        top3[1][j][0][:2],
        top3[2][k][0][:2],
        top3[3][l][0][:2],
        top3[0][i][1]
        + top3[1][j][1]
        + top3[2][k][1]
        + top3[3][l][1]) for i, j, k, l in product(range(TOP_K), repeat=4)]
    combs_sorted = sorted(combs, key=lambda x: x[4], reverse=True)
    _did_not_find = True
    for i, j, k, l, score in combs_sorted[:6]:
        anss = [i, j, k, l]
        if no_find_counter > 5:
            anss = ['fm' if c == 'cr' else c for c in anss]
        print(f"{anss}, score={score}")
        idxs = [flags.index(ans+'.png') for ans in anss]
        res = s.post(f'{target}/solve', data={'ans[]': anss})
        res = res.json()
        if res['message'] == 'Correct!':
            solved = res['solves']
            p, i = crack_p_m(idxs, base64.b64decode(token['checksum'].encode()), base64.b64decode(token['nonce'].encode()))
            crt_mods.append(p)
            crt_rems.append(i)
            sec = crt_solve(crt_mods, crt_rems)
            print("solved", solved, "of", total, "p", p, "i", i, "current recovered=", sec)
            if _last_rec == sec and sec > 2**500:
                solved_key = sec
                print("FOUND KEY", solved_key)
            _last_rec = sec
            print()
            _did_not_find = False
            break
    if _did_not_find:
        print("did not find any solution")
        no_find_counter += 1
    else:
        no_find_counter = 0

flag = s.get(f'{target}/flag/0').content
with open('flag.gif', 'wb') as f:
    f.write(flag)
```

*You need around 63 pairs of cid and salt to solve the key.*

```
...
flag 0: py.png  0.8836 | lu.png  0.8816 | nl.png  0.8526
flag 1: fk.png  0.7427 | ai.png  0.7162 | ky.png  0.6691
flag 2: at.png  0.8692 | hu.png  0.7367 | ye.png  0.7323
flag 3: gw.png  0.9318 | gn.png  0.4459 | bf.png  0.4453
['py', 'fk', 'at', 'gw'], score=3.427266836166382
solved 62 of 100 p 241 i 3 current recovered= 9034580482079416097353360839094264117402457854708698071398441424161722739946752015503046631972236900369507753109516677869604908568473586815049901944748569

flag 0: tv.png  0.6915 | nz.png  0.5581 | ms.png  0.5542
flag 1: ro.png  0.9162 | md.png  0.9160 | ad.png  0.9090
flag 2: uy.png  0.7367 | il.png  0.2699 | lt.png  0.1864
flag 3: gy.png  0.7826 | er.png  0.5207 | st.png  0.4751
['tv', 'ro', 'uy', 'gy'], score=3.126911163330078
['tv', 'md', 'uy', 'gy'], score=3.126708447933197
solved 63 of 100 p 577 i 158 current recovered= 9034580482079416097353360839094264117402457854708698071398441424161722739946752015503046631972236900369507753109516677869604908568473586815049901944748569
FOUND KEY 9034580482079416097353360839094264117402457854708698071398441424161722739946752015503046631972236900369507753109516677869604908568473586815049901944748569

['gm', 'sv', 'fr', 'tk'], score=3.126708447933197
{'message': 'Correct!', 'solves': 64, 'total': 100, 'imp': 0}
['sa', 'nf', 'zm', 'np'], score=3.126708447933197
{'message': 'Correct!', 'solves': 65, 'total': 100, 'imp': 0}
['cm', 'pf', 'kw', 'nr'], score=3.126708447933197
{'message': 'Correct!', 'solves': 66, 'total': 100, 'imp': 0}
...
['er', 'md', 'la', 'gq'], score=3.126708447933197
{'message': 'Correct!', 'solves': 98, 'total': 100, 'imp': 1}
['bg', 'gh', 'lt', 'dz'], score=3.126708447933197
{'message': 'Correct!', 'solves': 99, 'total': 100, 'imp': 1}
['az', 'tn', 'ae', 'as'], score=3.126708447933197
{'message': 'Correct!', 'solves': 100, 'total': 100}
```

![flag.gif](https://res.cloudinary.com/kumonochisanaka/image/upload/v1753694311/20250728051829737.gif/4d3132e3a57d302efad6890be99b350c.gif)

```flag
wwf{throughout_heaven_and_earth}
```

Honestly I think someone who can solve this solo is really impressive.
