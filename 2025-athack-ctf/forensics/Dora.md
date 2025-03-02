---
created: 2025-03-01T16:38
updated: 2025-03-01T16:50
---

```
binwalk -e Jungle.jpeg

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
406084        0x63244         Zip archive data, at least v2.0 to extract, name: treasureChest/
406160        0x63290         Zip archive data, at least v2.0 to extract, uncompressed size: 270158, name: treasureChest/Recording1.wav
612659        0x95933         Zip archive data, at least v2.0 to extract, uncompressed size: 368, name: __MACOSX/treasureChest/._Recording1.wav
613027        0x95AA3         Zip archive data, at least v2.0 to extract, uncompressed size: 306552, name: treasureChest/Recording2.wav
626070        0x98D96         Zip archive data, at least v2.0 to extract, uncompressed size: 611, name: __MACOSX/treasureChest/._Recording2.wav

WARNING: One or more files failed to extract: either no utility was found or it's unimplemented
```

`Recording2` has beep boops.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1740865230/2025/03/e660a92b28ae2a7e58a80cc1ffe97ea2.png)

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1740865554/2025/03/0895fbceab2a075d0e97298f72223103.png)

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1740865584/2025/03/66f6c87cf0845d7843d52f05c6b5a4ee.png)

It is a spectrogram.

```
DWKDFNFWI BRX_IRXQG_WK3_FRRUGLQDW3V
```

Looks like rotation cipher.

| Method     | Result                                 |
| ---------- | -------------------------------------- |
| [A-Z]+3    | `ATHACKCTF{YOU_FOUND_TH3_COORDINAT3S}` |
| [A-Z0-9]+3 | `ATHACKCTF{8OU_FOUND_TH0_COORDINAT0S}` |

```flag
ATHACKCTF{YOU_FOUND_TH3_COORDINAT3S}
```

It is.
