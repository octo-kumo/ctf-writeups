# Unknown File

```text
89504E470D0A1A0A0000000D4948445200000370000002BC ...
```

The attachment seems like a hex encoded file, let's decode it

```text
.PNG ...
```

Seems like a PNG image to me
![](image.png)

But where is the flag?

By changing the height to `968` and updating the CRC, we reveal the true image

```text
89504E47 0D0A1A0A 0000000D 49484452 00000370 000002BC 08060000 00AEB75D AC00000C
                                                  |||          |||||||| ||||||||
89504E47 0D0A1A0A 0000000D 49484452 00000370 000003B6 08060000 00A95B75 7E00000C
```

![](image.extended.png)

```text
CDDC22{S6oW_me_y0u're_4he_8est}
```