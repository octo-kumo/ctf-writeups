---
created: 2024-07-21T02:54
updated: 2024-07-21T16:07
tags:
  - audio
solves: 19
points: 471
---

## analysis
Upon opening the file, we can see a suspicious line on the spectrogram at 8000Hz, only in one channel.
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721545498/2024/07/8987985ec534194f6de159409b59f81f.png)

### basic filters
Invert the second channel and combine.
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721544912/2024/07/ce2aea6678d61a9812cab76602346b32.png)

The line seems to be carrying data, since the flag length is 39, we can guess how much each character takes. (the screenshot below was on a slowed track)

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721544934/2024/07/c3935a6489adee42586807ed40430fdb.png)

It does seem to carry binary info, take note that `ictf` has binary `01101001 01100011 01110100 01100110` and the peaks vaguely line up with the binary, that's great, we are on the right track.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721544896/2024/07/def5acd8050b58a8438926f695c08860.png)
However no matter what filter I try audacity just wouldn't give me good results.

## programmatic analysis

### sanity check
Simple sanity check, repeating what we did before with the channel difference.

```python
audio = AudioSegment.from_mp3("morning_n_breakfast.mp3")
left_channel_array = np.array(audio.split_to_mono()[0].get_array_of_samples())
right_channel_array = np.array(audio.split_to_mono()[1].get_array_of_samples())
data = left_channel_array-right_channel_array
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721544979/2024/07/e6c03c05721252f1e4e821cfe86283d4.png)

Great! the details are much more pronounced than in Audacity.

### band pass noise reduction
We use a band pass filter at 8000hz to remove noise.

```python
def butter_bandpass(lowcut, highcut, fs, order=5):
  nyq = 0.5 * fs
  low = lowcut / nyq
  high = highcut / nyq
  b, a = signal.butter(order, [low, high], btype='band')
  return b, a
def butter_bandpass_filter(data, lowcut, highcut, fs, order=5):
  b, a = butter_bandpass(lowcut, highcut, fs, order=order)
  y = signal.lfilter(b, a, data)
  return y
frequency = 8000
bandwidth = 200
filtered_data = butter_bandpass_filter(data, frequency-bandwidth, frequency+bandwidth, audio.frame_rate, order=5)
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721545019/2024/07/971812c454b06c9cc7f8156276588490.png)

Seems like each character take 8000 samples, nice.
Each character can also be divided into 9 sections first 8 being the bits and last being separator.

Great let's decode the flag...

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721546009/2024/07/d5d5b30ab7e53d94c5602bde031c43db.png)

Oh... misalignment issues.
### window size search
Since 8000 is not the correct character window size, I began searching for the correct window size by comparing the decoded output of 30th and 40th characters.

Each separate sub chart is the sub window of that bit, so my goal is to move the peaks to their edge, aligning all sub windows to bit changes.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721545044/2024/07/6f69f74eb01dfc074508483d3c07b131.png)
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721545055/2024/07/5c7bcf726b57203f2cf50de7dc5e32b0.png)

By manually searching through the window sizes, reducing search bounds and repeat, I have arrived at the optimal window size, 7941.

> In hindsight I should probably use a gap finding algorithm that will search for that 9th "separator" section.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721545129/2024/07/3df7bc5db1faaf08c3cbaee56f469286.png)

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1721545196/2024/07/8c27b793a5142ef377cc4656772861cc.png)

Hooray!

```
ictf{1_sl33py_NaYuKi_alarm_2a634dc1bb2}
```

## solve script

I removed the chart drawing parts for simplicity sake.
AI helped a lot, I hope they can fix my sleeping schedule in the future.

```python
from pydub import AudioSegment
import numpy as np
import scipy.signal as signal

audio = AudioSegment.from_mp3("morning_n_breakfast.mp3")
left_channel_array = np.array(audio.split_to_mono()[0].get_array_of_samples())
right_channel_array = np.array(audio.split_to_mono()[1].get_array_of_samples())
data = left_channel_array-right_channel_array


def butter_bandpass(lowcut, highcut, fs, order=5):
    nyq = 0.5 * fs
    low = lowcut / nyq
    high = highcut / nyq
    b, a = signal.butter(order, [low, high], btype='band')
    return b, a


def butter_bandpass_filter(data, lowcut, highcut, fs, order=5):
    b, a = butter_bandpass(lowcut, highcut, fs, order=order)
    y = signal.lfilter(b, a, data)
    return y


frequency = 8000
bandwidth = 200
filtered_data = butter_bandpass_filter(data, frequency-bandwidth, frequency+bandwidth, audio.frame_rate, order=5)


def extract(i, W=7941):
    sec = filtered_data[i*W:(i+1)*W]
    w = len(sec) // 9
    sec = [
        np.abs(sec[int(i*w + 0.25*w): int((i+1)*w - 0.25*w)])  # for less noise
        # np.abs(sec[int(i*w) : int((i+1)*w)]) # for debug
        for i in range(9)
    ]
    b = [np.mean(s) > 60 for s in sec[:8]]
    return sec, chr(np.packbits(np.array(b, dtype=np.uint8))[0])


for i in range(40):
    result, char = extract(i)
    print(char, end="")
```
