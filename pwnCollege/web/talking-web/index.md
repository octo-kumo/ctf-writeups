---
created: 2024-12-31T18:16
updated: 2025-01-01T14:50
---

Why am I doing web challenges on the pwn site?

## level 1

```bash
curl localhost
```

```flag
pwn.college{YeH-903cU1Un03tHQ2RZa7zJLO_.dhjNyMDL4MTNzYzW}
```

## level 2

```bash
nc localhost 80
GET /
```

```flag
pwn.college{4-JJP7vVo8RvzmZdXjlxjcaSOnZ.dljNyMDL4MTNzYzW}
```

## level 3

```python
>>> import urllib.request
>>> urllib.request.urlopen("http://localhost").read()
```

```flag
pwn.college{I5h1rCCSCuJpRjIpRuzJ0bF0f_p.dBzNyMDL4MTNzYzW}
```

## level 4

```bash
$ curl -H 'Host: b04f193cd403676badbdbe739cfb8462' localhost
```

```flag
pwn.college{Q6cl1PvHshDadxPdzOGTZ0gILWF.dFzNyMDL4MTNzYzW}
```

## level 5

```bash
$ nc localhost 80
GET /
Host: 932a28c5e24831163827d253a05812f0
```

```flag
pwn.college{UqAfeYHpEV0_SNUoXG_Bh74u_VD.dJzNyMDL4MTNzYzW}
```

## level 6

```python
import urllib.request
urllib.request.urlopen(urllib.request.Request("http://localhost", headers={"Host": "c92060ff1ec1da2dcd2682e93de20151"})).read()
```

```flag
pwn.college{4q5VzKoqFz-E6JlDXKYe3Vtj2hb.dNzNyMDL4MTNzYzW}
```

## level 7

```bash
$ curl localhost/fd848767631be50a163449a6dee105ce
```

```flag
pwn.college{8ThehfTKZA0cUhuKULD4YQG8_sL.dRzNyMDL4MTNzYzW}
```

## level 39

```python
import requests
print(requests.get("http://localhost").text)
```

```flag
pwn.college{0IMLYFAb5mo7eSTpn5eCXZ4G_R0.dZDMzMDL4MTNzYzW}
```
