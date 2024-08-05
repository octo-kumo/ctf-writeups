---
created: 2024-08-04T06:07
updated: 2024-08-04T21:20
points: 100
---

Empty password is not safe.

```python
@app.route('/<uid>')
def user_page(uid):
    if uid != str(uuid.uuid5(leet, 'admin123')):
        return f'Welcome! No flag for you :('
    else:
        return flag
# str(uuid.uuid5(leet, 'admin123')) = 3c68e6cc-15a7-59d4-823c-e7563bbb326c
```

```flag
n00bz{1337-13371337-1337-133713371337-1337}
```
