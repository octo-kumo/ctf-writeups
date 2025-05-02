---
ai_date: '2025-04-27 05:26:56'
ai_summary: Recovered audio data from distorted image using cumulative sum on binary
  array
ai_tags:
- imgf
- audio
- diff
created: 2025-01-04T21:36
points: 494
solves: 12
tags:
- dpcm
- delta-modulation
updated: 2025-01-06T23:22
---

Was solved after tons of guessing.

The title `DΔς` was the biggest hint.

![chal.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736045275/2025/01/250216429e95ddda913361e57f4119a8.png)

```
artifact title: A-199530

description:
  when a photo is taken with artifact in frame,
  photo is distorted and contains what looks like a
  barcode, left to right, top to bottom. however,
  it is not a standard barcode and appears to encode
  a long stream of bits, either being negative or
  positive. additionally, the message appears to be
  some type of audio.

additional notes from lab:
  ABCD = [1, 0, 1, -1
          1, 1, 1, -2
          0, 1, 0,  0]
  order = 2
```

The audio wave form is stored as the *difference*, so to recover it we will use `np.cumsum`.

```python
from PIL import Image
import numpy as np
image = Image.open('chal.png').convert('L')
img_array = np.array(image)
threshold = 128
binary_array = (img_array > threshold).astype(np.int8)
rows = binary_array[980:2835:7, :]
columns = rows[:, :]

from scipy.io.wavfile import write
from IPython.display import Audio

decoded_data = columns.flatten()
decoded_data = np.cumsum(decoded_data * 2 - 1)
scaled_data = np.int16(decoded_data * (32767 / np.max(np.abs(decoded_data))))
sample_rate = 44100
write("output.wav", sample_rate, scaled_data)
Audio("output.wav")
```

```flag
irisctf{a_signal_in_the_cave}
```

The audio caused me permanent ear damage.