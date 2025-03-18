---
created: 2025-03-01T15:33
updated: 2025-03-18T02:26
solves: 30
points: 50
---

It's just a password zip file.

```bash
$ zip2john flag.zip > hash.txt
Created directory: /home/kali/.john
ver 1.0 efh 5455 efh 7875 flag.zip/flag.txt PKZIP Encr: 2b chk, TS_chk, cmplen=58, decmplen=46, crc=584F77B1 ts=8A28 cs=8a28 type=0

$ john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 16 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
welostjester     (flag.zip/flag.txt)
1g 0:00:00:00 DONE (2025-03-01 15:33) 3.846g/s 10838Kp/s 10838Kc/s 10838KC/s wendreen0601..wardkiel
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

```flag
ATHACKCTF{j000n_c0nGr44tzzz_dis_iz_ur_fl4ggg}
```
