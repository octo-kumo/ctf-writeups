---
created: 2026-03-08T05:07
points: 80
solves: 22
title: Past of the internet?
updated: 2026-03-08T15:12
---

My teammates searched everywhere for it but turns out it was in plain sight.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1772997148/20260308151227775.png/1d1e4aed797fb57fdd2c9c0866fe67c2.png)

Only two snapshots were made in 2026, and on the same day too.

```bash
curl -L "https://web.archive.org/web/20260216053447id_/https://hexploit-alliance.com/" -o version_1.gz
curl -L "https://web.archive.org/web/20260216205751id_/https://hexploit-alliance.com/" -o version_2.gz
gunzip version_1.gz
gunzip version_2.gz
diff version_1 version_2
```

And the flag, in plaintext too.

```
26c26
<
---
>     <!-- ATHACKCTF{7h3_p457_4nd_7h3_pr353n7} -->
```

```flag
ATHACKCTF{7h3_p457_4nd_7h3_pr353n7}
```