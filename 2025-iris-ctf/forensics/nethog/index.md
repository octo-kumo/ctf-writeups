---
created: 2025-01-05T06:07
updated: 2025-01-05T19:06
solves: 3
points: 500
---

> I hate it when I have a bad link and my large data transfer gets corrupted. Apparently this weird new protocol is supposed to help with that, but I think the developers skipped an important step...
> **Hint!** I do not like the bits that fail, But with this code, you will prevail! Fix the errors, it’s not so tough— With `********`, you’ll have enough!

Loading the `.pcap` in Wireshark does not give any useful results.

So I loaded it in `pyshark` to be examined.

- Each packet is $512$ bytes long.

First discovery was that there is some sort of counter at the front.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736075421/2025/01/e302efacb7bee94ad9cb7d4d355f4400.png)

From the hints we can guess that this has to do with error checking codes, some fixing errors.
I spent hours on hours, trying to figure out what's been used, guessing parities, only to fail.

At some point I decided for fun, to just draw all the bits as an image.

![output_image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736075688/2025/01/f0c1d4ccfabd0c6e98c57aabdc130d14.png)

Oh well, well well.

It appears that each packet is in fact, 4 packets!

Let's reshape the image.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736075784/2025/01/6f006d815a39982014739065a2a53e3f.png)

We can notice that the "counter" from before actually have 2 more bits.

At this point I got stuck again due to sleep deprivation and just brain not functioning.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736075911/2025/01/a45e0608790084000d4fcefea1a96a40.png)

Then I saw the lines, they were located at powers of 2 locations, I asked the almighty AIs and they told me this is the sign of **Hamming Code**.

## Hamming Code

Hamming codes are a family of linear error-correcting codes that can detect and correct **single-bit errors**.

They achieve this by adding *parity* bits to *data* bits.

In Hamming codes, parity bits are placed at positions that are **powers of 2** (e.g., 1, 2, 4, 8, 16, etc., for 1-indexed systems, or 0, 1, 2, 4, 8, etc., for 0-indexed systems).

## Solve

Apply correction to each sub-packet, then remove the first 16 counting bites.

```python
import numpy as np
import pyshark
from PIL import Image


def bytes_to_8bit_str(b):
    return ' '.join(format(byte, '08b') for byte in b)


def bit_array_to_bytes(bit_array):
    return bytes(int(''.join(map(str, bit_array[i:i + 8])), 2) for i in range(0, len(bit_array), 8))


def decode_hamming(bits):
    syndrome = 0
    parity_positions = {0}.union({2**i for i in range(0, 10)})
    for pos in range(10):
        pos = 2**pos
        parity = sum(bits[j] for j in range(pos, 1024, pos*2) for k in range(j, min(j + pos, 1024)) if k not in parity_positions) % 2
        if parity != 0:
            syndrome |= pos
    error_position = None
    if syndrome != 0:
        error_position = syndrome
        if error_position >= len(bits):
            raise ValueError("Too many errors detected")
        else:
            bits[error_position] ^= 1
    return [bits[i] for i in range(len(bits)) if i not in parity_positions]


pcap_file = 'drone.pcap'
packets = pyshark.FileCapture(pcap_file)
image = []
img2 = []
decoded = []
for pkt in packets:
    payload = bytes.fromhex(pkt.udp.payload.replace(':', ''))
    if (len(payload) != 512):
        payload = payload.ljust(512, b'\x00')
    payload = list(map(int, list(''.join(format(byte, '08b') for byte in payload))))
    image.append(payload)
    p1, p2, p3, p4 = payload[:1024], payload[1024:2048], payload[2048:3072], payload[3072:]
    for p in [p1, p2, p3, p4]:
        data = decode_hamming(p)
        img2.append(data)
        decoded.extend(data[16:])

decoded = bit_array_to_bytes(decoded)

with open('output.jpg', 'wb') as f:
    f.write(decoded)

image = np.array(image)
img2 = np.array(img2)

image_data = np.array([[int(bit) for bit in row] for row in img2], dtype=np.uint8)
img = Image.fromarray(image_data * 255)
img.save('output_1.png')

image_data = np.array([[int(bit) for bit in row] for row in image], dtype=np.uint8)
img = Image.fromarray(image_data * 255)
img.save('output_image.png')

image_data = image_data.reshape(-1, 1024)
img = Image.fromarray(image_data * 255)
img.save('output_image_cols.png')
```

> Side note: during the last stage, I actually messed up by converting bits to bytes a bit early inside the loop, causing the JPG to be unreadable and I panicked for a moment lmao.

![output.jpg](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736076580/2025/01/839164a8a912ecfdb7cc365c5b2bea7c.jpg)

```flag
irisctf{th4t_w4snt_too_h4rd}
```

This is the error corrected bit stream.

![output_1.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736076750/2025/01/fd1dac180a230863c85233135464ae94.png)
