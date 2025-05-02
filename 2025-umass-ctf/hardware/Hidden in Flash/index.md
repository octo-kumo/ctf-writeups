---
ai_date: '2025-04-27 05:27:36'
ai_summary: Discovered SWF file hidden in firmware extraction, used JPEXS decompiler
  to extract flag
ai_tags:
- swf
- decompilation
- exploitation
created: 2025-04-18T23:21
points: 499
solves: 7
title: Hidden in Flash
updated: 2025-04-20T20:26
---

My teammate extract the firmware but it wasn't exactly right.

So I did my own extraction.

## extraction
Took a few attempts for the I2C to actually work, GPT for some reason refuse to believe that you should send the lower bits first then the upper bits.

```c [eeprom_dump.ino]
#include <Wire.h>

const uint8_t  EE_ADDR = 0x54;
const uint32_t START = 0;
const uint32_t TOTAL = 4096;

void setup() {
    Wire.begin();
    Serial.begin(115200);
    delay(100);
}

void loop() {
    for (uint32_t addr = START; addr < TOTAL; addr += 1) {
        Wire.beginTransmission(EE_ADDR);
        Wire.write(uint8_t(addr & 0xFF));
        Wire.write(uint8_t(addr >> 8));
        Wire.endTransmission(false);
        uint8_t  chunk = 1;
        Wire.requestFrom(EE_ADDR, chunk);
        for (int i = 0; i < chunk; i++) {
            while (!Wire.available()) delayMicroseconds(1);
            Serial.write(Wire.read());
        }
    }
    while (1);
}
```

## automation
Slight modification of the `client.py` given to make it modify the C source to pull 4096 bytes each time and then reconstruct the full firmware.

```python
import os
import re
import sys
import subprocess
import socket
import time

START = 0
TOTAL = 4 * 1024
HOST = "hardware.ctf.umasscybersec.org"
PORT = 10003
BUILD_DIR = "build"


def update_bounds(sketch_name="eeprom_dump.ino"):
    path = os.path.join('.', sketch_name)
    with open(path, 'r') as f:
        code = f.read()

    new_line = f"const uint32_t TOTAL   = {TOTAL};\n"
    pattern = r"const\s+uint\d+_t\s+TOTAL\s*=.*?;"
    updated_code, count = re.subn(pattern, new_line.strip(), code)
    if count == 0:
        print("WARNING: No TOTAL definition found")

    new_line = f"const uint32_t START   = {START};\n"
    pattern = r"const\s+uint\d+_t\s+START\s*=.*?;"
    updated_code, count = re.subn(pattern, new_line.strip(), updated_code)
    if count == 0:
        print("WARNING: No START definition found")

    with open(path, 'w') as f:
        f.write(updated_code)
    print(f"Updated TOTAL&START in {sketch_name}.")


def compile_sketch():
    cmd = [
        "arduino-cli", "compile",
        "--fqbn", "arduino:avr:uno",
        "--build-path", 'build',
        '.'
    ]
    print("Compiling sketch...")
    try:
        subprocess.run(cmd, check=True)
    except subprocess.CalledProcessError as e:
        print(f"Compilation failed: {e}")
        sys.exit(1)

    build_path = os.path.join('.', BUILD_DIR)
    return os.path.join(build_path, "eeprom_dump.ino.elf")


def send_firmware(elf_path):
    with open(elf_path, 'rb') as f:
        data = f.read()

    time_err = TimeoutError("Did not receive expected data in time.")

    def recv(sock, num_bytes, timeout=5.0):
        output = b''
        start = time.time()
        while num_bytes > 0 and time.time() - start < timeout:
            recvd = sock.recv(num_bytes)
            if not recvd:
                break
            num_bytes -= len(recvd)
            output += recvd
        if num_bytes:
            raise time_err
        return output

    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        print("Connecting...")
        s.connect((HOST, PORT))
        print("Sending firmware...")
        s.sendall(len(data).to_bytes(4, 'little') + data)
        if recv(s, 1) != b"\x00":
            print("Unknown response from server")
            sys.exit(1)

        print("Running code...")
        rsp_msgs = [
            "Code ran successfully!",
            "Internal error setting up sim."
            "The sim crashed while running your code."
        ]
        ret = int.from_bytes(recv(s, 1), 'little')
        if ret < len(rsp_msgs):
            print(rsp_msgs[ret])
        else:
            print("Unknown response from server")
        out_len = int.from_bytes(recv(s, 4), 'little')
        data = recv(s, out_len)
        return data


def main():
    global TOTAL, START
    all_data = b''
    total_iterations = 16
    for i in range(total_iterations):
        START = i * 1024*4
        TOTAL = (i + 1) * 1024*4
        print(f"Trying with START={START} and TOTAL={TOTAL}")
        update_bounds()
        elf = compile_sketch()
        data = send_firmware(elf)
        all_data += data
        print(f"Received {len(data)} bytes of data.")
        print(f"{i}/{total_iterations}")
    print("Writing data to eeprom_dump.bin")
    with open("eeprom_dump.bin", "wb") as f:
        f.write(all_data)
    print("Done!")


if __name__ == '__main__':
    main()
```

```
00000000  43 57 53 06 88 9F 00 00  78 9C E4 BD 07 58 14 CD  CWS.....x....X..
00000010  D2 30 DA B3 89 05 96 9C  A3 48 06 C9 48 10 50 16  .......H..H.P.
00000020  24 E7 9C 24 C3 92 24 09  0B 82 71 41 45 50 54 40  $..$..$...qAEPT@
...
```

```bash
$ file eeprom_dump.bin
eeprom_dump.bin: Macromedia Flash data (compressed), version 6
```

It is a SWF file!
## the flag

I used [jindrapetrik/jpexs-decompiler: JPEXS Free Flash Decompiler](https://github.com/jindrapetrik/jpexs-decompiler).

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1745033202/2025/04/fb5292ad30ad86a5bdd091579ae61f4b.png)

```flag
UMASS{asT3r0iDs!1}
```

> Fun fact, I didn't realise theres more than 8KB because GPT was convinced it has only 8KB, so I didn't find the flag at first, which led me to crack a MD5 password inside the swf.

```bash
# passwordHash : String = $1$Ir$m0pQBLDgXyF9aZ9JcoH/v0
$ john --show --format=md5crypt hash.txt
?:112365
```