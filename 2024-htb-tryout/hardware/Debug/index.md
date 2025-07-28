---
ai_date: 2025-04-27 05:16:11
ai_summary: Flag found by analyzing serial data with estimated frequency of 115741 Hz
ai_tags:
  - logic2
  - serial-communication
  - freq-estimation
created: 2024-07-17T01:41
points: 975
solves: 71
tags:
  - sal
updated: 2025-07-14T09:46
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