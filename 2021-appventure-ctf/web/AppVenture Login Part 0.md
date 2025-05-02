---
ai_date: '2025-04-27 05:10:15'
ai_summary: Flag found in robots.txt disallowed path pointing to source code
ai_tags:
- robots.txt
- src-code
- 信息披露
created: 2021-12-21T23:50
updated: 2024-06-10T23:38
---

## AppVenture Login Part 0

> AppVenture Login page must be the most secure right? URL: http://35.240.143.82:4208/

Hint:

> What's the first thing you do when pentesting a website?

One of the common files that websites contain is the `robots.txt`, which decides what scrapers like google-bot can see and should see.

In this case the robots contains a path to the source code of the website, and the flag is inside the source code.

### http://35.240.143.82:4208/robots.txt

```
User-agent: *
Disallow: /c7179ef35b2d458d6f2f68044816e145/main.py
```

### http://35.240.143.82:4208/c7179ef35b2d458d6f2f68044816e145/main.py

```
...
flag0 = "flag{you_can_use_automated_tools_like_nikto_to_do_this}"
...
```

Flag obtained