---
created: 2024-06-11T01:17
updated: 2024-07-07T23:08
---

```text
89504E470D0A1A0A0000000D4948445200000370000002BC ...
```

The attachment seems like a hex encoded file, let's decode it

```text
.PNG ...
```

Seems like a PNG image to me

![](https://res.cloudinary.com/kumonochisanaka/image/upload/v1718083777/2024/06/b68bd42ed31e1eb40289486096281220.png)

But where is the flag?

By changing the height to `968` and updating the CRC, we reveal the true image

```text
89504E47 0D0A1A0A 0000000D 49484452 00000370 000002BC 08060000 00AEB75D AC00000C
                                                  |||          |||||||| ||||||||
89504E47 0D0A1A0A 0000000D 49484452 00000370 000003B6 08060000 00A95B75 7E00000C
```

![](https://res.cloudinary.com/kumonochisanaka/image/upload/v1718083777/2024/06/63ef2516ae1946b5dd238729506a0c24.png)

```flag
CDDC22{S6oW_me_y0u're_4he_8est}
```
