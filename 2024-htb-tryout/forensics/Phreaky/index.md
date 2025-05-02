---
ai_date: '2025-04-27 05:16:02'
ai_summary: Decompressing and combining SMTP email attachments to reveal a PDF file
ai_tags:
- smtp
- email
- decomp
- pdf
created: 2024-07-17T00:16
points: 950
solves: 265
updated: 2024-08-04T19:30
---

I extracted files from HTTP but they are all useless xz archives.

Then I realized that there are suspicious SMTP packages, and I saw those emails.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721189964/2024/07/24ac259bd4c43d95a5d64848016e7e76.png)

I just have to decompress all of the chunks, and put them together.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721191306/2024/07/b2db186b9f31edee450a07abce7fb507.png)

## Solve Script

```python
import base64
import pathlib
import re
import zipfile
import io

dirname = pathlib.Path(__file__).parent.resolve()
pdfbuf = b''
for i in range(15):
    with open(f"{dirname}/Secure File Transfer({i}).eml", "r") as f:
        email = ''.join(f.readlines())
    password = re.search(r'Attached is a part of the file. Password: (\w+)', email).group(1)
    b64str = re.search(r'Content-Type: application/zip[\s\S]+Content-ID: .+\n([^-]+)', email).group(1).strip()
    zipbuf = base64.b64decode(b64str.encode())
    z = zipfile.ZipFile(io.BytesIO(zipbuf))
    z.setpassword(password.encode())
    for zfile in z.filelist:
        pdfbuf += z.read(zfile.filename)
with open(f"{dirname}/Secure File Transfer.pdf", "wb") as f:
    f.write(pdfbuf)

# HTB{Th3Phr3aksReadyT0Att4ck}
```