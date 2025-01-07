---
created: 2025-01-05T04:21
updated: 2025-01-06T23:20
solves: 26
points: 470
tags:
  - fm
---

My teammate @zeptoide basically solve it but the audio file was too fast, so I just slowed it down.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736122217/2025/01/80653ec0032ecc35ea748ad3bf675351.png)

```python
from pydub import AudioSegment

audio = AudioSegment.from_wav("pls.wav")
slowed_audio = audio._spawn(audio.raw_data, overrides={
    "frame_rate": int(audio.frame_rate * 0.3)
})
slowed_audio = slowed_audio.set_frame_rate(audio.frame_rate)
slowed_audio.export("pls_slowed.wav", format="wav")
```

```flag
irisctf{grc_is_great_for_simple_narrowband_modulation}
```