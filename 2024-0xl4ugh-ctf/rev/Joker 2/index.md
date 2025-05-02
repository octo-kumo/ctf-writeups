---
ai_date: '2025-04-27 05:12:31'
ai_summary: Malware encodes zip file in 'Would you lose.png' using XOR encryption
  with a 4-byte key; solved by extracting and decrypting the image
ai_tags:
- xor
- imgf
- decryption
created: 2024-12-27T23:30
points: 440
solves: 7
updated: 2024-12-28T06:27
---

The malware seems to be fetching a bunch of files and putting them into "Exfiltrated_data.zip" then "Would you lose.png".

We can guess that "Would you lose.png" contains data since that's the only file we were given.
## main

Looking at the code, after all the file operations, there is a function call to `sub_140001480(v31, v37)` where:

- `v31` contains the path to "Exfiltrated_data.zip"
- `v37` contains the path to "Would you lose.png"

```c [somewhere above]
if ( (int)sub_140001060(v31, 0x104ui64, "%s\\Exfiltrated_data.zip") < 260 )
  {
    if ( (int)sub_140001060(v37, 0x104ui64, "%s\\Would you lose.png") >= 260 )
```

## image generation
Looking into `sub_140001480` we find the function that encodes the zip into the image.

1. It reads the input file (the zip file)
2. XORs the data
    - Uses `BCryptOpenAlgorithmProvider` to generate random bytes
    - Creates a 4-byte random key (falls back to `rand_s` if `BCrypt` fails)
    - XORs the entire file content with this 4-byte key
3. Creates a header
	- CRC32 checksum (4 bytes)
	- The encryption key (stored as 8 bytes?)
	- Original data size (4 bytes)
4. Generates the image

## solve

Solving it is hence simple, just use the provided key.

```python
import png
import struct
import zlib
import zipfile

png_path = "Would you lose.png"
output_path = "out.zip"
reader = png.Reader(filename=png_path)
width, height, pixels, metadata = reader.read()
pixel_data = []
for row in pixels:
    for pixel in zip(*[iter(row)]*4):
        pixel_data.extend(pixel)
pixel_data = bytes(pixel_data)
stored_crc, key, orig_size = struct.unpack("<IQI", pixel_data[:16])
calc_crc = zlib.crc32(pixel_data[4:16])
if calc_crc != stored_crc:
    raise "CRC mismatch, data is corrupted"

encrypted_data = pixel_data[16:16+orig_size]
key_bytes = key.to_bytes(8, 'little')[:4]
decrypted_data = bytearray()
for i, byte in enumerate(encrypted_data):
    decrypted_data.append(byte ^ key_bytes[i % 4])
with open(output_path, 'wb') as f:
    f.write(decrypted_data)
with zipfile.ZipFile(output_path, 'r') as zip_ref:
    zip_ref.extractall("extracted_files")
```

![Stand_Proud_You_Are_Strong.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1735361259/2024/12/7a65b4ee07cc4ad1e3be82d3ed985cd3.jpg)

```flag
0xL4ugh{I_4lon3_4m_Th3_H0n0r3d_0ne_641}
```