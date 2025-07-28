---
ai_date: 2025-04-27 05:23:34
ai_summary: Suspicious image size revealed hidden width; discovered using SOF0 marker and length
ai_tags:
  - imgf
  - len-ext
  - magic-bytes
created: 2024-06-22T05:38
points: 182
solves: 118
tags:
  - cropped-image
updated: 2025-07-14T09:46
---

The image is suspiciously large for its pixel size.

 > Look for the magic bytes `ff c0`

Found them. `ff c0 00 11 08 00 0a 00 0a`

| `ff c0`       | `00 11` | `08`      | `00 0a` | `00 0a` |
| ------------- | ------- | --------- | ------- | ------- |
| `SOF0` marker | length  | precision | height  | width   |

Turns out the correct width is `00 a0`.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1719049731/2024/06/037531ce8e921f5de2ab1b0569667353.png)

```flag
FLAG{b1g_en0ugh}
```