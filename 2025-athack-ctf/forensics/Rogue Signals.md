---
ai_date: 2025-04-27 05:25:06
ai_summary: DNS query revealed zip file containing a password-protected PDF; cracked with 'hackme' password.
ai_tags:
  - dns-enum
  - zip
  - pdf
created: 2025-03-01T15:41
points: 100
solves: 20
updated: 2025-07-14T09:46
---

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1740862660/2025/03/a14901ba5f149f54ef5a90f0743d4d35.png)

The first DNS request is a dead give away, this is a zip file header.

```python
import pyshark
from tqdm import tqdm

cap = pyshark.FileCapture('Rogue_Signals.pcapng')
all_hexs = b''
for pkt in tqdm(cap):
    if 'dns' in pkt:
        name = pkt.dns.qry_name
        if '.athackctf.com' in name:
            hexs = name.split('.')[0]
            all_hexs += bytes.fromhex(hexs)

with open('output.zip', 'wb') as f:
    f.write(all_hexs)
```

## zip

After unzipping the zip file, we get a txt file and a password protected zip.

``` [important.txt]
Hello Defender!

If you arrived here, you are already a 1337.


Have you ever seen a password protected PDF file !


What's your next move ?


The flag is waiting for YOUUUUUUUUU !!!
```

```bash
$ pdf2john secret_protected.pdf > file.hash
$ john --wordlist=/usr/share/wordlists/rockyou.txt file.hash
Using default input encoding: UTF-8
Loaded 1 password hash (PDF [MD5 SHA2 RC4/AES 32/64])
Cost 1 (revision) is 3 for all loaded hashes
Will run 16 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
hackme           (secret_protected.pdf)     
1g 0:00:00:00 DONE (2025-03-01 15:59) 2.857g/s 277942p/s 277942c/s 277942C/s jessica101..ericalynn
Use the "--show --format=PDF" options to display all of the cracked passwords reliably
Session completed.
```

Password is `hackme`.

```flag
ATHACKCTF{$_D4TA_ExF1L7r47i0N_Via_DNS_Ch4LL3nG3_4cc3p7ed_$}
```