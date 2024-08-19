---
created: 2024-08-08T09:04
updated: 2024-08-17T20:13
points: 225
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
