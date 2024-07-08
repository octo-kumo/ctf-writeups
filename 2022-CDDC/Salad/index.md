---
created: 2024-06-11T01:17
updated: 2024-07-07T23:08
---

Using exiftool or just hex editor, we can find a `tEXt` chunk near the end of the image

```text
00000000 00000000 e8754e7f 2c206e41
2cee395d b892255d ccbf7e5d 4cec4441
64c52c7f 58ac7000 d0af3667 e8be2a1c
d4b39907 e4f6a566 24426e5f e0223460

94875307 f4bbc15c 18ec1607 ecb71b0a
44a5cc3d e0127666 bc2c0b67 00642600
fc55557f 04ad4c41 7419025d 7415685d
74a55a5d 04bd7241 fc69757f 00000000
```

Since the salad image has some big grids, lets try converting this to a QR code with its binary representation

```python
import base64

import matplotlib.pyplot as plt
import matplotlib.cm as cm
import numpy as np

import cv2

src = """
00000000 00000000 e8754e7f 2c206e41
2cee395d b892255d ccbf7e5d 4cec4441
64c52c7f 58ac7000 d0af3667 e8be2a1c
d4b39907 e4f6a566 24426e5f e0223460

94875307 f4bbc15c 18ec1607 ecb71b0a
44a5cc3d e0127666 bc2c0b67 00642600
fc55557f 04ad4c41 7419025d 7415685d
74a55a5d 04bd7241 fc69757f 00000000
"""

data = list(bytes.fromhex(src))
print(len(data))
qr = []
for i in range(0, 128, 4):
    row = ""
    for j in range(4):
        row += "{0:08b}".format(data[i + j])[::-1]
    qr.append([int(k) == 0 for k in row])
plt.imsave('generated_qr.png', np.rot90(np.array(qr), 2).repeat(50, axis=0).repeat(50, axis=1), cmap=cm.gray)

detector = cv2.QRCodeDetector()
data, _, _ = detector.detectAndDecode(cv2.imread("generated_qr.png"))
print(data)
data = base64.b64decode(data)
print(data)
print(''.join([chr(ord(i) - 3) if i != "{" and i != "}" and i != "_" else i for i in data.decode()]))
```

![](https://res.cloudinary.com/kumonochisanaka/image/upload/v1718083689/2024/06/146e7f37237c90b39e603944355d8cac.png)

```text
128
RkdHRjU1ezhkT2RnOF9EdTZfajMzZ183X2s2ZG93a30=
b'FGGF55{8dOdg8_Du6_j33g_7_k6dowk}'
CDDC22{5aLad5_Ar3_g00d_4_h3alth}
```
