---
created: 2025-08-02T07:59
updated: 2025-08-02T08:01
---

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1754136059/20250802080058804.png/d8b7482f3bfd07e54a251d6526de44ec.png)

```python
import numpy as np
from scipy.io import wavfile
import matplotlib.pyplot as plt

rate, data = wavfile.read("morse1.wav")
data = np.abs(data)
data = np.array([np.max(data[i:i+1000]) for i in range(0, len(data), 1000)])
data[data < 10000] = 0
data[data > 0] = 1

b_data = ''

c = data[0]
count = 1

for value in data[1:]:
    if value == c:
        count += 1
    else:
        rep = round(count/10)
        b_data += str(c) * rep
        c = value
        count = 1
        
bytes_list = [int(b_data[i:i+8], 2) for i in range(0, len(b_data), 8)]
print(bytes(bytes_list))
```

```flag
justCTF{The-track-name-is-Coldness-and-the-artist-is-The-Wanderer}
```
