---
created: 2024-12-27T18:12
updated: 2024-12-28T06:26
solves: 5
points: 460
---

Trying to decode the binary was a bit too difficult for me, so I decided to brute force the flag.

I discovered that when encrypting $N$ bytes, the first $N$ output bytes are always the same, further more the first byte of every 10 bytes is also always the same (no matter the byte itself).

Below are the encrypted values of `'aaaaaaaaaabbbbbbbbbbcccccccccc'[:i]`.

We can observe a very clear pattern.

```text
1     a1 26 d1 3a 04 ee 66 55 9e 98
2     a1 0d 97 a4 e7 c3 63 31 4c 99
3     a1 0d 8a 26 8f 59 6b 58 b0 92
4     a1 0d 8a b6 92 36 3f 58 fd 93
5     a1 0d 8a b6 bd 6a 4f f4 16 94
6     a1 0d 8a b6 bd bc 36 0b c4 95
7     a1 0d 8a b6 bd bc 06 69 a7 8e
8     a1 0d 8a b6 bd bc 06 07 15 8f
9     a1 0d 8a b6 bd bc 06 07 b1 90
10    a1 0d 8a b6 bd bc 06 07 b1 b0 bb 10 1e bd 50 9a de 49 2c ea
11    a1 0d 8a b6 bd bc 06 07 b1 b0 bb f4 19 55 2a b2 f3 4f e0 ed
12    a1 0d 8a b6 bd bc 06 07 b1 b0 bb b7 0c 71 4e f1 66 f3 47 ec
13    a1 0d 8a b6 bd bc 06 07 b1 b0 bb b7 0d 25 a8 ba 04 62 ef df
14    a1 0d 8a b6 bd bc 06 07 b1 b0 bb b7 0d 8c 29 a6 4d 15 af de
15    a1 0d 8a b6 bd bc 06 07 b1 b0 bb b7 0d 8c 87 fd 5a 42 49 e1
16    a1 0d 8a b6 bd bc 06 07 b1 b0 bb b7 0d 8c 87 bb 3a 2e 8c e0
17    a1 0d 8a b6 bd bc 06 07 b1 b0 bb b7 0d 8c 87 bb 09 42 61 e3
18    a1 0d 8a b6 bd bc 06 07 b1 b0 bb b7 0d 8c 87 bb 09 08 95 e2
19    a1 0d 8a b6 bd bc 06 07 b1 b0 bb b7 0d 8c 87 bb 09 08 83 e5
20    a1 0d 8a b6 bd bc 06 07 b1 b0 bb b7 0d 8c 87 bb 09 08 83 82 c5 12 fb 7a 19 48 31 2f b0 ea
21    a1 0d 8a b6 bd bc 06 07 b1 b0 bb b7 0d 8c 87 bb 09 08 83 82 c5 21 27 20 9e aa 26 3a 84 ed
22    a1 0d 8a b6 bd bc 06 07 b1 b0 bb b7 0d 8c 87 bb 09 08 83 82 c5 0b 55 e5 42 3d 37 0f 00 ec
23    a1 0d 8a b6 bd bc 06 07 b1 b0 bb b7 0d 8c 87 bb 09 08 83 82 c5 0b 39 22 30 53 f5 61 2d df
24    a1 0d 8a b6 bd bc 06 07 b1 b0 bb b7 0d 8c 87 bb 09 08 83 82 c5 0b 39 b8 95 a0 31 64 63 de
25    a1 0d 8a b6 bd bc 06 07 b1 b0 bb b7 0d 8c 87 bb 09 08 83 82 c5 0b 39 b8 bb 10 4d 51 d0 e1
26    a1 0d 8a b6 bd bc 06 07 b1 b0 bb b7 0d 8c 87 bb 09 08 83 82 c5 0b 39 b8 bb 87 1e 0b ca e0
27    a1 0d 8a b6 bd bc 06 07 b1 b0 bb b7 0d 8c 87 bb 09 08 83 82 c5 0b 39 b8 bb 87 3d 37 94 e3
28    a1 0d 8a b6 bd bc 06 07 b1 b0 bb b7 0d 8c 87 bb 09 08 83 82 c5 0b 39 b8 bb 87 3d 09 53 e2
29    a1 0d 8a b6 bd bc 06 07 b1 b0 bb b7 0d 8c 87 bb 09 08 83 82 c5 0b 39 b8 bb 87 3d 09 82 e5
```

This means we can brute force byte by byte since the first $N$ output bytes are guaranteed to be equal as long as the first $N$ input bytes are equal.

```python
def enc(data):
    f = f"tmp/test{random.randint(1000000, 10000000)}.txt"
    with open(f, 'wb') as tmp_file:
        tmp_file.write(data)
    subprocess.run(['./vault', f], stdout=subprocess.DEVNULL)
    with open(f+'.enc', 'rb') as tmp_file:
        output_data = tmp_file.read()
    try:
        os.remove(f)
        os.remove(f+'.enc')
    except:
        pass
    return output_data
```

## BFS

I am unsure of how the search will be branching, I don't even know if the outputs are guaranteed to be unique.

So I first ran BFS on it.

```python
def get_flag_bfs(queue):
    while queue:
        curr = queue.popleft()
        print(curr, queue)
        pos = get_char(curr, double=True)
        for c in pos:
            _curr = curr + c.encode()
            queue.append(_curr)
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1735354058/2024/12/8371f7d3ba5e85acce23ff63f638b34e.png)

Turns out the chars are mostly the same, and the branch singles out (no alternative branch) frequently.

Which means I can speed up the search drastically using DFS on a reduced charset.

## DFS

Will try to search as far as possible, before encountering a stop.

```python
with open('flag.txt.enc', 'rb') as f:
    flag_enc = f.read()

reducedP = ' $|_/\\\n\''
printable = reducedP
for c in string.printable:
    if c not in reducedP:
        printable += c

doubleP = [''.join(p) for p in itertools.product(printable, repeat=2)]

def get_char(curr, double=False, reduced=False):
    pos = []

    def check_char(c):
        _curr = curr + c.encode()
        e = enc(_curr)
        if e[:len(_curr)] == flag_enc[:len(_curr)]:
            return c
        return None

    if double and len(curr) % 10 == 0:
        with ThreadPoolExecutor() as executor:
            results = list(tqdm(executor.map(check_char, doubleP), total=len(doubleP), desc=f"len: {len(curr)}"))
    else:
        with ThreadPoolExecutor() as executor:
            results = list(executor.map(check_char, reducedP if reduced else printable))

    pos = [c for c in results if c is not None]
    return pos

def get_flag_dfs(start, brave=False):
    global best
    stack = [start]
    while stack:
        curr = stack.pop()
        if len(curr) > len(best):
            best = curr
            if len(best) % 10 == 0:
                with open('flag.txt', 'wb') as f:
                    f.write(best)
        print(curr.decode())
        if len(curr) % 10 == 0 and not brave:
            pos = list(printable)
        else:
            pos = get_char(curr, reduced=True)
            if not pos and not brave:
                pos = get_char(curr)
        stack.extend(curr + c.encode() for c in pos)

if os.path.exists('flag.txt'):
    with open('flag.txt', 'rb') as f:
        curr = f.read()
else:
    curr = b''

get_flag_dfs(curr, True)
```

Manual intervention is required once in a while to turn on brave mode for fast but unsafe searching, and also adding new characters to the reduced charset.

It takes a few minute to decode the top part, and I ate dinner while it decoded the last parts in complete safe mode.

```text
'    /$$$$$$                                               /$$                     /$$/$$/$$
'   /$$__  $$                                             | $$                    | $| $| $$
'  | $$  \__/ /$$$$$$ /$$$$$$$  /$$$$$$  /$$$$$$ /$$$$$$ /$$$$$$ /$$$$$$$$/$$$$$$$| $| $| $$
'  | $$      /$$__  $| $$__  $$/$$__  $$/$$__  $|____  $|_  $$_/|____ /$$|____ /$$| $| $| $$
'  | $$     | $$  \ $| $$  \ $| $$  \ $| $$  \__//$$$$$$$ | $$     /$$$$/   /$$$$/|__|__|__/
'  | $$    $| $$  | $| $$  | $| $$  | $| $$     /$$__  $$ | $$ /$$/$$__/   /$$__/           
'  |  $$$$$$|  $$$$$$| $$  | $|  $$$$$$| $$    |  $$$$$$$ |  $$$$/$$$$$$$$/$$$$$$$$/$$/$$/$$
'   \______/ \______/|__/  |__/\____  $|__/     \_______/  \___/|________|________|__|__|__/
'                              /$$  \ $$                                                    
'                             |  $$$$$$/                                                    
'                              \______/                                                     


'   /$$   /$$                                  /$$                /$$     /$$                               /$$$$$$$$/$$                  /$$/$$
'  | $$  | $$                                 |__/               |  $$   /$$/                              | $$_____| $$                 | $| $$
'  | $$  | $$ /$$$$$$  /$$$$$$  /$$$$$$        /$$ /$$$$$$$       \  $$ /$$/$$$$$$ /$$   /$$ /$$$$$$       | $$     | $$ /$$$$$$  /$$$$$$| $| $$
'  | $$$$$$$$/$$__  $$/$$__  $$/$$__  $$      | $$/$$_____/        \  $$$$/$$__  $| $$  | $$/$$__  $$      | $$$$$  | $$|____  $$/$$__  $| $| $$
'  | $$__  $| $$$$$$$| $$  \__| $$$$$$$$      | $|  $$$$$$          \  $$| $$  \ $| $$  | $| $$  \__/      | $$__/  | $$ /$$$$$$| $$  \ $|__|__/
'  | $$  | $| $$_____| $$     | $$_____/      | $$\____  $$          | $$| $$  | $| $$  | $| $$            | $$     | $$/$$__  $| $$  | $$      
'  | $$  | $|  $$$$$$| $$     |  $$$$$$$      | $$/$$$$$$$/          | $$|  $$$$$$|  $$$$$$| $$            | $$     | $|  $$$$$$|  $$$$$$$/$$/$$
'  |__/  |__/\_______|__/      \_______/      |__|_______/           |__/ \______/ \______/|__/            |__/     |__/\_______/\____  $|__|__/
'                                                                                                                                /$$  \ $$      
'                                                                                                                               |  $$$$$$/      
'                                                                                                                                \______/       
__
                  / \--..____
                   \ \       \-----,,,..
                    \ \       \         \--,,..
                     \ \       \         \  ,'
                      \ \    0xL\         \ ``..
                       \ \       \4ugh     \-''
                        \ \       \__,,--'''
                         \ \       \.
                          \ \      ,/
                           \ \__..-
                            \ \
                             \ \
                              \ \  0xL4ugh{r1se_and_r1se_ag@1n_unt1l_l@mbs_b3c0me_l1ons_e099c665_0b}
                               \ \ 
                                \ \
                                 \ \
                                  \ \
                                   \ \
                                    \ \
                           -----------------------
#+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
#                                                                                                                                          +
#
```

```flag
0xL4ugh{r1se_and_r1se_ag@1n_unt1l_l@mbs_b3c0me_l1ons_e099c665_0b}
```
