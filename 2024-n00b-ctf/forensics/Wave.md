---
ai_date: 2025-04-27 05:19:58
ai_summary: Morse code hidden in corrupted .wav file, solved using a Morse code decoder.
ai_tags:
  - audio
  - morse-code
  - stegano
created: 2024-08-04T06:12
description: Fixing corrupted wav file
points: 387
solves: 267
tags:
  - wav
updated: 2025-07-14T09:46
---

## Corrupted `.wav` file.
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1722755514/2024/08/9f547fadf25bc1ea21e55bb78f2cc0ab.png)

## Sample wave file
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1722755523/2024/08/1c601bf38ad55009a0f28412d02f3b73.png)
Hey at least they did not remove the size info, and the sizes are correct.

## Restored
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1722755726/2024/08/aa3bde13299afdfeded1935b8dc397e4.png)
## Solve
It's morse code. [Morse Code Adaptive Audio Decoder | Morse Code World](https://morsecode.world/international/decoder/audio-decoder-adaptive.html)

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1722755766/2024/08/8b2e427c72bdd97c0f88bff0b569b44f.png)

```flag
n00bz{beepbopmorsecode}
```