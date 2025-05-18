---
ai_date: 2025-05-17 21:38:34
ai_summary: CSV parsing reveals hidden characters in LED Backpack code, hinting at ASCII art XOR cipher
ai_tags:
  - xor
  - ascii-art
  - ciphertext
created: 2025-05-17T05:47
points: 477
solves: 58
title: Alpha
updated: 2025-05-17T22:01
---

First we export the data to CSV via Logic 2.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1747475861/2025/05/8cf348e909be33fb93f0e5aada82299c.png)

Then we parse it.

```python
import csv


def load_charset(cpp_path):
    cmap = {}
    with open(cpp_path, encoding='utf-8', errors='ignore') as f:
        in_table = False
        ascii_idx = 0
        for line in f:
            if not in_table:
                if 'static const PROGMEM uint16_t alphafonttable' in line:
                    in_table = True
                continue
            if '};' in line:
                break
            parts = line.split('//')
            code = parts[0].strip().rstrip(',')
            if not code:
                continue
            if ',' in code:
                continue
            mask = int(code, 0)
            ch = None
            if len(parts) > 1 and parts[1].strip():
                ch = parts[1].strip()[0]
            else:
                if ascii_idx == 32:
                    ch = ' '
            if ch:
                cmap[mask] = ch
            ascii_idx += 1
    return cmap


def parse_i2c(csv_path):
    recs = []
    buf = b''
    with open(csv_path, newline='') as f:
        reader = csv.DictReader(f)
        for row in reader:
            if row['name'] != 'I2C':
                continue
            if row['data']:
                buf += bytes([int(row['data'], 16)])
            if row['type'] == 'stop':
                if buf:
                    recs.append(buf)
                buf = b''
    return recs


if __name__ == '__main__':
    import sys
    csv_path, cpp_path = sys.argv[1], sys.argv[2]

    cmap = load_charset(cpp_path)
    cmap[0] = ''
    records = parse_i2c(csv_path)
    lines = [[] for _ in range(4)]
    for r in records:
        print(r.hex(), len(r))
        if len(r) >= 16:
            for i in range(4):
                mask = int.from_bytes(r[1+i * 2:1+i * 2 + 2], 'little')
                if mask in cmap:
                    lines[i].append(cmap[mask])
                else:
                    print(f'Unknown mask: {mask:016b} {mask:04x}')
                    lines[i].append('?')
    for i in range(4):
        print(''.join(lines[i]))
```

Get char map from [Adafruit_LEDBackpack.cpp](https://github.com/adafruit/Adafruit_LED_Backpack/blob/master/Adafruit_LEDBackpack.cpp).

```bash
python solve.py byuctf.csv Adafruit_LEDBackpack.cpp
```

For some reason, the best I could get was this, I had to guess the last character.

```
byuctf{4r3n7_h4rdw4r3_pr070c0l5_c0?byuct
yuctf{4r3n7_h4rdw4r3_pr070c0l5_c0??yuctf
uctf{4r3n7_h4rdw4r3_pr070c0l5_c00??uctf{
ctf{4r3n7_h4rdw4r3_pr070c0l5_c00l??ctf{4
```

And it wasn't that bad.

```flag
byuctf{4r3n7_h4rdw4r3_pr070c0l5_c00l?}
```