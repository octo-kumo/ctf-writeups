---
created: 2025-01-05T04:22
updated: 2025-01-05T19:09
points: 359
solves: 56
---

By the sheer amount of IQ radio challenges in this CTF, I can literally guess it is IQ.

The noisy-af ear-shredding sounds can't stop me, I used an online noise remover AI.

## solve

```python
import numpy as np
from scipy.io.wavfile import write
from scipy.signal import resample
from pwn import *
from tqdm import tqdm
r = remote('rfoip-620ac7b1.radio.irisc.tf', 6531)
chunks = b''
for i in tqdm(range(10000)):
    chunks += r.recv()
with open('output.raw', 'wb') as f:
    f.write(chunks)

with open("output.raw", "rb") as f:
    chunks = f.read()
iq_samples = np.frombuffer(chunks, dtype=np.int16)
i_samples = iq_samples[::2]
q_samples = iq_samples[1::2]
complex_samples = i_samples + 1j * q_samples

phase = np.angle(complex_samples)
demodulated_signal = np.diff(phase)
demodulated_signal = demodulated_signal / np.max(np.abs(demodulated_signal))
audio_sample_rate = 48000
num_samples = int(len(demodulated_signal) * audio_sample_rate / 80000)
resampled_signal = resample(demodulated_signal, num_samples)
resampled_signal = np.clip(resampled_signal, -1.0, 1.0)
write("output.wav", audio_sample_rate, resampled_signal.astype(np.float32))
print("WAV file saved: output.wav")
```

```flag
irisctf{welcome_to_iris_radio_enjoy_surfing_the_waves}
```
