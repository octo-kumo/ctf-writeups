---
ai_date: 2025-05-17 21:38:44
ai_summary: Flag found in XML element positions after decompiling APK
ai_tags:
  - xml
  - decomp
  - android
created: 2025-05-17T06:52
points: 241
solves: 191
title: Baby Android 1
updated: 2025-05-17T21:52
---

First decompile the APK using `apktool`.

```bash
apktool d baby-android-1.apk
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1747479406/2025/05/17289875b583f6f2befe70589396cd2d.png)

And it is right there.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1747480022/2025/05/7ac73347a1c2678ea4accde6266e894b.png)

```python
import matplotlib.pyplot as plt
import xml.etree.ElementTree as ET
tree = ET.parse('flag.xml')
root = tree.getroot()

chars = []
for elem in root.iter():
    mB = elem.get('{http://schemas.android.com/apk/res/android}layout_marginBottom')
    mE = elem.get('{http://schemas.android.com/apk/res/android}layout_marginEnd')
    if mB is None or mE is None:
        continue
    mB = float(mB.replace('dip', ''))
    mE = float(mE.replace('dip', ''))
    chars.append((elem.get("{http://schemas.android.com/apk/res/android}text"), mB, mE))


sortedByB = sorted(chars, key=lambda x: x[1], reverse=True)
sortedByE = sorted(chars, key=lambda x: x[2], reverse=True)
print("".join([x[0] for x in sortedByB]))
print("".join([x[0] for x in sortedByE]))


plt.figure(figsize=(10, 10))
for char, mB, mE in chars:
    plt.text(mE, mB, char)
plt.grid(True)
plt.xlabel('MarginEnd')
plt.ylabel('MarginBottom')
plt.title('Character Positions')
plt.xlim(0, 1000)
plt.ylim(0, 1000)
plt.show()
```

```flag
byuctf{android_piece_0f_c4ke}
```