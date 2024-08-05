---
created: 2024-06-22T02:44
updated: 2024-08-05T09:51
solves: 431
points: 126
---

Using hex editor we can see that the file starts with `RDP8bmp`, with some google search we know that it is a BMP cache file.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1719038693/2024/06/9280f42ea811742502668c4423126539.png)

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1719038670/2024/06/a9b5549538e9b93a3ef5cc56b831b4bd.png)

```flag
FLAG{RDP_is_useful_yipeee}
```

- [BSI-Bund/RdpCacheStitcher: RdpCacheStitcher is a tool that supports forensic analysts in reconstructing useful images out of RDP cache bitmaps. (github.com)](https://github.com/BSI-Bund/RdpCacheStitcher)
- [ANSSI-FR/bmc-tools: RDP Bitmap Cache parser (github.com)](https://github.com/ANSSI-FR/bmc-tools)
