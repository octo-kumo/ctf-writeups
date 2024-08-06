---
created: 2024-06-22T05:38
updated: 2024-08-05T19:29
solves: 118
points: 182
tags:
  - cropped-image
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
