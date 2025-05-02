---
ai_date: '2025-04-27 05:27:41'
ai_summary: Flag found by decoding a binary signal using OOK demodulation and pattern
  recognition of repeating bits
ai_tags:
- demod
- pattern-recognition
- binary-decoding
created: 2025-04-20T13:19
points: 497
solves: 10
tags:
- rf
- demodulation
- iq
title: RF Fan
updated: 2025-04-20T20:26
---

My teammate @zeptoide already did much of the work of writing the solve script and also finding the parameters.

He was able to find this, but couldn't figure out the answer to it.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1745169893/2025/04/2fffdccf193b95b6b448597b9ece9021.png)

Well I stared at it and realised that it could just be 7 lines of binary, where each bit is sent a few times repeatedly. Which means the dimmer regions are 1 and the brighter regions are 0.

To illustrate this, let's convert the file into a bit image.

![output.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1745169828/2025/04/b560918001833837a8159ce46c510411.png)

And add some separation.

![output.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1745170334/2025/04/bd752e7e2b3282ba89b98f4e3d10164d.png)

```
01010001001111111011100000110010
01010001001111111011100001000101
01010001001111111011100001010100
01010001001111111011100001100111
01010001001111111011100001110110
01010001001111111011100000000001
01010001001111111011100000010000
```

We can see a clear pattern of common prefix `0101000100111111101110000` $+k_{i}\times\{0\}+k_i\times \{ 1 \}$, and if you look closer its just a binary counter and a 3 bit number, all numbers except 3 has appeared.

So the last line would be.

```
0110010 / 2
1000101 / 5
1010100 / 4
1100111 / 7
1110110 / 6
0000001 / 1
0010000 / 0

0100011 / 3
```

If we order them based on the counter we get packets `1 0 3 2 5 4 7 6`.

And combined with the header we have this.

```
01010001001111111011100000100011
```

Which is the flag.

```flag
UMASS{01010001001111111011100000100011}
```

## script

By teammate @zeptoide.

```bash
python3 hmm.py --input signal.iq --bits-out bits.bin --frames-out frames.txt --dtype complex64 --decim 800 --min-gap 50 --use-squared
```

```python [hmm.py]
import numpy as np
import argparse
import os

def parse_args():
    p = argparse.ArgumentParser(description="OOK demod & frame extractor")
    p.add_argument("--input",    "-i", required=True, help="Input IQ file (raw interleaved)")
    p.add_argument("--bits-out", "-b", required=True, help="Raw bitstream output (0x00/0x01 bytes)")
    p.add_argument("--frames-out","-f",required=True, help="Extracted frames output (text, one per line)")
    p.add_argument("--dtype",    "-d", default="complex64",
                   choices=["complex64","complex128"], help="NumPy dtype of IQ samples")
    p.add_argument("--decim",    "-D", type=int, default=1, help="Decimation factor (samples per bit)")
    p.add_argument("--min-gap",  "-g", type=int, default=20, help="Min consecutive zeros to delimit frames")
    p.add_argument("--use-squared", action="store_true",
                   help="Use magnitude-squared for envelope instead of magnitude")
    return p.parse_args()

def extract_frames(bits, min_gap=20):
    frames = []
    n = len(bits)
    i = 0
    while i < n:
        # Skip until a '1' is found
        while i < n and bits[i] == 0:
            i += 1
        if i >= n:
            break
        start = i
        
        zero_count = 0
        j = i
        # Scan until min_gap zeros in a row
        while j < n:
            if bits[j] == 0:
                zero_count += 1
            else:
                zero_count = 0
            if zero_count >= min_gap:
                end = j - zero_count + 1
                break
            j += 1
        else:
            end = n
        
        frame_bits = bits[start:end]
        frames.append(''.join(str(b) for b in frame_bits))
        
        # Advance past this gap
        i = j
        while i < n and bits[i] == 0:
            i += 1
    return frames

def main():
    args = parse_args()

    # 1) Load IQ data
    iq = np.fromfile(args.input, dtype=args.dtype)
    if iq.size == 0:
        raise RuntimeError(f"No data read from {args.input}")

    # 2) Envelope detection
    env = np.abs(iq)
    if args.use_squared:
        env = env**2

    # 3) Threshold slicing
    thr = (env.max() + env.min()) / 2
    bits = (env > thr).astype(np.uint8)

    # 4) Decimate to 1 sample/bit
    if args.decim > 1:
        bits = bits[::args.decim]

    # 5) Save raw bits
    bits.tofile(args.bits_out)
    print(f"Wrote raw bitstream ({bits.size} samples) to {args.bits_out}")

    # 6) Extract frames
    frames = extract_frames(bits, min_gap=args.min_gap)
    print(f"Extracted {len(frames)} frames using gap ≥{args.min_gap} zeros")

    # 7) Save frames to text
    with open(args.frames_out, 'w') as f:
        for frame in frames:
            f.write(frame + '\n')
    print(f"Wrote frames to {args.frames_out}")

if __name__ == "__main__":
    main()
```

Converter script.

```python [convert.py]
from PIL import Image
import numpy as np

# Read all lines from frames.txt
with open('frames.txt', 'r') as f:
    lines = [line.strip() for line in f.readlines()]

# Calculate average line length
avg_length = sum(len(line) for line in lines) / len(lines)

# Duplicate shorter lines 8 times
width = max(len(line) for line in lines)
processed_lines = []
for line in lines:
    if len(line) < avg_length:
        processed_lines.extend(['0'*width]*4)  # add 8 rows of 0
    processed_lines.append(line)

# Update lines with processed_lines
lines = processed_lines

# Find the maximum length (width) of binary strings
width = max(len(line) for line in lines)
height = len(lines)

# Create a numpy array to store the image data
image_data = np.zeros((height, width), dtype=np.uint8)

# Convert binary strings to image data
for y, line in enumerate(lines):
    for x, bit in enumerate(line):
        # Convert '1' to white (255) and '0' to black (0)
        image_data[y, x] = 255 if bit == '1' else 0

# Create and save the image
image = Image.fromarray(image_data)
image = image.resize((width * 2, height * 2), Image.NEAREST)
image.save('output.png')
```