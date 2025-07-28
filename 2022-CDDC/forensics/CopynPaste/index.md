---
ai_date: 2025-04-27 05:10:50
ai_summary: SVG animation reveals base64-encoded PNG flag after parsing and decoding
ai_tags:
  - svg
  - base64-decode
  - png
created: 2024-06-11T01:17
title: Copy & Paste
updated: 2025-07-14T09:46
---

The svg is an animation of a terminal window it seems, well a bit of digging and we find this

![](https://res.cloudinary.com/kumonochisanaka/image/upload/v1718083133/2024/06/72db4692a297c19c5a3f3f4fff0a7ba7.png)

Looks pretty simple I would say

```python
import xml.etree.ElementTree as ET
import base64

mytree = ET.parse('copy_n_paste.svg')
myroot = mytree.getroot()
text = ''
appending = False
for svg in myroot[1][1]:
    for s1 in svg:
        if s1.text == 'iVBORw0KGgoAAAANSUhEUgAABOwAAAFmCAYAAADeVgWcAAAKBWlDQ1BJQ0MgUHJvZmlsZQAASImVlndU':
            appending = True
        if s1.tag == '{http://www.w3.org/2000/svg}text' and appending:
            text += s1.text
        if s1.text == 'dNG/BgAAAABJRU5ErkJggg==':
            appending = False
print(text)

fh = open("copy_n_paste", "wb")
fh.write(base64.b64decode(text))
fh.close()
```

With this we will get the out file

![](https://res.cloudinary.com/kumonochisanaka/image/upload/v1718083134/2024/06/dfee8ea1be6b0a798b4ab9abe72a7b85.png)

It is an PNG image, adding the extension gives us

![](https://res.cloudinary.com/kumonochisanaka/image/upload/v1718083135/2024/06/a7ef03dca2f7c6adcfd8656a8066cc24.png)

```flag
CDDC22{S4V4G3_LOVE}
```