---
ai_date: '2025-04-27 05:12:15'
ai_summary: Extracted flag from filtered signal using FFT analysis and peak detection,
  with adjustments for weak first bit
ai_tags:
- fft
- freq-analysis
- peak-detection
created: 2024-12-27T17:06
points: 450
solves: 6
updated: 2024-12-28T06:30
---

Trace file that seems pretty well structured, 7 bits, solved with FFT and a bunch of filters.

```python
import numpy as np
import matplotlib.pyplot as plt

data=np.load("trace.npy")
plt.plot(data)
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1735337485/2024/12/2fa3d8448ffcce10137d46eeffb52b21.png)

## FFT

We first have to determine the frequency on which data is carried, we can do that with Fast Fourier Transform.

```python
Fs = 800
fft_data = np.fft.fft(data)
frequencies = np.fft.fftfreq(len(data), d=1/Fs)
plt.figure(figsize=(10, 5))
plt.plot(frequencies[:len(frequencies)//2], np.abs(fft_data)[:len(frequencies)//2])
plt.xlabel("Frequency (Hz)")
plt.ylabel("Magnitude")
plt.title("Frequency Domain Analysis")
y = np.abs(fft_data)[:len(frequencies)//2]
start_index = 4000
end_index = 10000
sub_array = y[start_index:end_index + 1]
relative_max_index = np.argmax(sub_array)
best_freq = frequencies[:len(frequencies)//2][start_index + relative_max_index]
plt.vlines(best_freq, 0, np.max(y), colors='r', linestyles='dashed', label='Best Frequency')
plt.legend()
plt.show()
print(best_freq)
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1735337500/2024/12/627360630ac356684e7288e583a2042a.png)

The frequency is 200Hz (dependent on frame size used).
## extract 200hz

```python
from scipy.signal import savgol_filter
from scipy.signal import butter, filtfilt

time = np.arange(len(data)) / Fs
guess_freq = best_freq
low_cutoff = guess_freq-10
high_cutoff = guess_freq+10
b, a = butter(4, [low_cutoff / (Fs / 2), high_cutoff / (Fs / 2)], btype='band')
filtered_signal = filtfilt(b, a, data)
plt.figure(figsize=(20, 5))
plt.plot(time, filtered_signal, label=f"Filtered Signal ({guess_freq} Hz)")
plt.xlabel("Time (s)")
plt.ylabel("Amplitude")
plt.title(f"Filtered Signal Around {guess_freq} Hz")
plt.legend()
plt.show()
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1735337511/2024/12/ee5cd9b2f49d365be6e3dfd9e88d9d60.png)

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1735337646/2024/12/3dd3cbdc058d802a843ab1e210a55d9c.png)

Binary of `0xL4` is `00110000 01111000 01001100 00110100`.

Which seems to match up pretty well, just flipped horizontally.

## smooth + peaks

The peaks aren't really aligned, some peaks are separated by ~80 while others by ~100, so indexing directly won't work.

```python
from scipy.signal import find_peaks

absed = np.abs(filtered_signal)**4
absed=savgol_filter(absed, 50, 2)
absed=savgol_filter(absed, 25, 2)
absed = np.pad(absed, 50)
peaks, _ = find_peaks(absed, height=0.0000025)
plt.figure(figsize=(20, 5))
plt.plot(absed)
plt.plot(peaks, absed[peaks], "x")
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1735337544/2024/12/79daf655951b5c2a1054eafc7dab7a3e.png)

## get flag

For some reason the first bit is always weak, so it is boosted by 1.2.

```python
thres = 1e-5*0.75
diffs = np.diff(peaks)
buf = ""
for i, d in zip(peaks, diffs):
    v = absed[i]
    if len(buf) == 0:
        v*=1.2
    v = v > thres
    buf += '1' if v else '0'
    if d >150:
        buf += "0"
        print(chr(int(buf[::-1], 2)), end="")
        buf = ""

print(chr(int('1'+buf[::-1], 2)), end="")
```

```flag
0xL4ugh{Squinting4SPA_Sucks}
```