---
created: 2024-07-17T01:41
updated: 2024-08-04T20:31
tags:
  - sal
solves: 71
points: 975
---

## SAL Files
First, install [Logic 2](https://www.saleae.com/pages/downloads).
Then, install `Baud rate estimate`.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721196852/2024/07/0494d3a2fc09322b78e8d0db4d593c37.png)
## Estimate Frequency
Shift clicking the first two high voltage points to estimate the frequency.
It is about 115741 Hz.
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721196276/2024/07/b46984fdf9d4dbd080aacd8f633cff81.png)

## Read Data
Click on **"Async Serial"** to add an new analyser.
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721196404/2024/07/18c8e82958086258bd040137bf2f5513.png)

## Get Flag
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721196696/2024/07/c6a86a04f1d2f8b8b67c985655a30bfc.png)

Switch to console view and we can see our flag.

```flag
HTB{547311173_n37w02k_c0mp20m153d}
```
