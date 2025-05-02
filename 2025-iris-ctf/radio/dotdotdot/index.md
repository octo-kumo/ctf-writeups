---
ai_date: '2025-04-27 05:27:03'
ai_summary: 'Extracted morse code from audio signal at 1515 Hz, decoded to reveal
  flag: irisctf{n01s3_g0t_n0th1ng_0n_my_m0rse}'
ai_tags:
- spectrogram
- freq-discovery
- morse-code
created: 2025-01-06T14:53
points: 50
solves: 150
tags:
- morse
- signal
updated: 2025-01-06T23:28
---

I solved this right after the CTF started, so I forgot that I've solved this and only now am I making the writeup.

First let's convert the iq into wav.

```bash
sox -e float -t raw -r 44100 -b 32 -c 1 dotdotdot.iq -t wav -e float -b 32 -c 1 -r 44100 out.wav --norm
```

Then draw the spectrogram.

```python
import librosa
import librosa.display
import matplotlib.pyplot as plt
y, sr = librosa.load("out.wav")
D = librosa.stft(y)
S_db = librosa.amplitude_to_db(np.abs(D), ref=np.max)
fig, ax = plt.subplots()
img = librosa.display.specshow(S_db, sr=sr, x_axis='time', y_axis='hz', ax=ax)
fig.colorbar(img, ax=ax, format="%+2.f dB")
ax.set(title='Spectrogram')
plt.show()
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736193604/2025/01/d87ecdd639c39e859cd6c7075e245309.png)

There appears to be signal stored at 1 specific frequency.

## Frequency Discovery

```python
import matplotlib.pyplot as plt
import numpy as np
from scipy.io import wavfile
from scipy.fft import fft
from sklearn.cluster import DBSCAN

sample_rate, data = wavfile.read('out.wav')
#FFT
fft_result = fft(data[:, 0] if data.ndim > 1 else data)
frequencies = np.fft.fftfreq(len(fft_result), 1/sample_rate)
threshold = np.max(np.abs(fft_result)) * 0.1
spikes = frequencies[(np.abs(fft_result) > threshold) & (frequencies >= 0)]
clustering = DBSCAN(eps=20, min_samples=1).fit(spikes.reshape(-1, 1))
clustered_frequencies = [spikes[clustering.labels_ == label].mean() for label in set(clustering.labels_)]
print("Clustered frequencies (Hz):", clustered_frequencies)

plt.plot(frequencies[:len(frequencies)//2], np.abs(fft_result)[:len(frequencies)//2])
plt.xlabel('Frequency (Hz)')
plt.ylabel('Amplitude')
plt.title('Frequency Spectrum')
for freq in clustered_frequencies:
    plt.axvline(x=freq, color='r', linestyle='--', label=f'{freq:f} Hz')
plt.legend()
plt.show()
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736197105/2025/01/e54eac149fb4c86b7b7dde1e4b458ca0.png)

I think it should be 1515Hz.

## Data Extraction

```python
from scipy.signal import butter, filtfilt
def bandpass_filter(data, lowcut, highcut, sample_rate, order=5):
    nyquist = 0.5 * sample_rate
    low = lowcut / nyquist
    high = highcut / nyquist
    b, a = butter(order, [low, high], btype="band")
    return filtfilt(b, a, data)
lowcut = 1515-50
highcut = 1515+50
filtered_signal = bandpass_filter(data, lowcut, highcut, sample_rate)
plt.figure(figsize=(10, 6))
plt.plot(filtered_signal[:100000])
plt.title("Filtered Signal (1515 Hz)")
plt.xlabel("Sample Index")
plt.ylabel("Amplitude")
plt.grid()
plt.show()
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736197426/2025/01/164634e468074feccc3ae724ed67162d.png)

Appears to be morse code.

## Morse Code

First let's filter the signal with ABS and smoothing.

```python
from scipy.signal import savgol_filter
from scipy.signal import find_peaks
import numpy as np
sig = np.abs(filtered_signal)
sig = savgol_filter(sig, 2001, 3, mode='nearest')
sig[sig < 0.1] = 0
sig[sig >= 0.1]=1
peaks, _ = find_peaks(sig)
plt.figure(figsize=(20, 5))
plt.plot(sig)
plt.xlim([0, 200000])
plt.plot(peaks, sig[peaks], "x")
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736197626/2025/01/8f17f43c82587fd9e99f0f6ea5a02228.png)

### Automatic Morse Decoder

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736199869/2025/01/4324e91aef0fa9ce585029716e4f0f2a.png)

```python
import numpy as np
from scipy.io import wavfile
from sklearn.cluster import KMeans

bit_stream = np.where(sig > 0.5, 1, 0)
edges = np.where(np.diff(bit_stream) != 0)[0]
durations = np.diff(edges) / sample_rate
states = bit_stream[edges[:-1]]
high_durations = durations[states == 0]
low_durations = durations[states == 1]
kmeans_high = KMeans(n_clusters=2).fit(high_durations.reshape(-1, 1))
dot_threshold = kmeans_high.cluster_centers_.flatten().mean()
kmeans_low = KMeans(n_clusters=3).fit(low_durations.reshape(-1, 1))
g1, g2, g3 = sorted(kmeans_low.cluster_centers_.flatten())
gt1 = (g1 + g2) / 2
gt2 = (g2 + g3) / 2
morse_code = []
for i, duration in enumerate(durations):
    if states[i] == 0:
        morse_code.append('.' if duration < dot_threshold else '-')
    else:
        morse_code.append(' _ ' if duration > gt2 else ' ' if duration > gt1 else '')
morse_dict = {
    '.-': 'A', '-...': 'B', '-.-.': 'C', '-..': 'D', '.': 'E', '..-.': 'F', '--.': 'G',
    '....': 'H', '..': 'I', '.---': 'J', '-.-': 'K', '.-..': 'L', '--': 'M', '-.': 'N',
    '---': 'O', '.--.': 'P', '--.-': 'Q', '.-.': 'R', '...': 'S', '-': 'T', '..-': 'U',
    '...-': 'V', '.--': 'W', '-..-': 'X', '-.--': 'Y', '--..': 'Z', '-----': '0',
    '.----': '1', '..---': '2', '...--': '3', '....-': '4', '.....': '5', '-....': '6',
    '--...': '7', '---..': '8', '----.': '9'
}
morse_message = ''.join(morse_code).split(' ')
decoded_message = ''.join([morse_dict.get(code, code) for code in morse_message])
print(decoded_message.lower())
# irisctf_n01s3_g0t_n0th1ng_0n_my_m0rse
```

```flag
irisctf{n01s3_g0t_n0th1ng_0n_my_m0rse}
```