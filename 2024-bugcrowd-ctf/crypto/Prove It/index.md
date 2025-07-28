---
ai_date: 2025-04-27 05:13:45
ai_summary: MD5 collisions demonstrated for file hashing, leading to a bypass of the security measure.
ai_tags:
  - md5
  - collisions
  - hash
created: 2024-08-08T09:04
points: 225
updated: 2025-07-14T09:46
---

MD5 isn't secure, finding collisions is easy.

```bash
echo "BHUSA24" > input
docker run --rm -it -v .:/work -w /work brimstone/fastcoll --prefixfile input -o msg1.bin msg2.bin
```

```python
import requests
print(requests.post("https://bugcrowd-prove-it.chals.io/upload", files={"file1": open("msg1.bin", "rb"), "file2": open("msg2.bin", "rb")}).text)
```

```flag
flag{use_bcrypt_you_salty_dog}
```