---
created: 2024-07-27T15:54
updated: 2024-07-28T02:14
tags:
  - audio
title: MAN in the middle
---

> Do you know that even the signals without encryption are vulnerable to the MAN in the middle attack?

## Analysis
The audio waveform is a bunch of square waves.
That certainly simplifies a lot of work for us.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1722110184/2024/07/a599c7fc4b6e5505bef6e64fc9175e08.png)

I will take a guess and say it could be Manchester, because naïve extraction of the above binary didn't give any usable data.

### Sample Width

```python
import matplotlib.pyplot as plt

time = np.arange(0, len(data))
plt.figure(figsize=(15, 3))
n = 7
p = 44
w = 10
plt.plot(time[p*n-w:p*n+w], data[p*n-w:p*n+w])
plt.axvline(x = p*n, color = 'b', label = 'center')
plt.title("Time Domain Signal")
plt.xlabel("Time (s)")
plt.ylabel("Amplitude")
plt.grid(True)
plt.show()
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1722110264/2024/07/774ae7c47955db5f11f7d4bc01303c20.png)

I determined the window to be length 44, each bit takes 44 samples.

### Extraction

```python
def extract(i):
  return 1 if data[i*44+5]>0 else 0

c = 0
char = 0
for i in range(0,len(data)//44,2):
  a = extract(i)^(1 if i%2==0 else 0)
  b = extract(i+1)^(1 if (i+1)%2==0 else 0)
  if a != b:
    print(i, a, b, "mismatch")
  char=(char<<1)|a
  c+=1
  if c==8:
    print(chr(char), end="")
    char=0
    c=0
```

**Manchester** signal works like this, so I would take two adjacent bits, flip one of them and check that they are now indeed equal, that would be the data bit.
![Manchester_encoding_both_conventions.svg](https://res.cloudinary.com/kumonochisanaka/image/upload/v1722111662/2024/07/f2454367aec82f0068f68f0365f774e5.svg)

```
manchester encoding is a method of encoding digital data in which each bit of data is represented by two voltage levels, ensuring a transition at the middle of each bit period. this transition serves as both a clock and data signal, making it highly effective for synchronous communication. developed by g. e. thomas, manchester encoding is widely used in various communication protocols, including ethernet. its primary advantage lies in its robustness against timing errors and ease of clock recovery, as the regular transitions enable the receiver to maintain synchronization with the transmitter. by embedding the clock signal within the data stream, manchester encoding mitigates the risk of synchronization loss, making it a reliable choice for high-speed digital data transmission. and here is your flag: dead{m4nch3573r_4_7h3_w1n} good job!
```

```flag
dead{m4nch3573r_4_7h3_w1n}
```

## Solve Script

```python
from pydub import AudioSegment
import numpy as np

audio = AudioSegment.from_wav("audio.wav")
data = np.array(audio.get_array_of_samples())


def extract(i):
    return (1 if data[i*44+5] > 0 else 0) ^ (1 if i % 2 == 0 else 0)


c = 0
char = 0
for i in range(0, len(data)//44, 2):
    a = extract(i)
    b = extract(i+1)
    if a != b:
        print(i, a, b, "mismatch")
    char = (char << 1) | a
    c += 1
    if c == 8:
        print(chr(char), end="")
        char = 0
        c = 0
```
