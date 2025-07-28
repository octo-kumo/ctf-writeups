---
ai_date: 2025-04-27 05:21:00
ai_summary: "Weak password vulnerability: Empty password 'admin123' allows unauthorized access."
ai_tags:
  - weak-pass
  - pwd-brute
  - xss
created: 2024-08-04T06:07
points: 100
solves: 736
updated: 2025-07-14T09:46
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