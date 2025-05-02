---
ai_date: '2025-04-27 05:18:48'
ai_summary: The writeup describes a challenge where the randomness of a circuit output
  is being tested, and a specific metric (np_entropy_mean) is used to exploit it.
  The success rate is higher than expected, suggesting a vulnerability in the randomness
  generation.
ai_tags:
- rand
- entropy
- exploitation
created: 2024-12-08T00:26
points: 119
solves: 86
updated: 2024-12-08T13:16
---

Output of the random circuit isn't all that random.

```python
if b == 0:
	res = random.getrandbits(B - SIZE)
else:
	res = b2i(eval_circuit(p, i2b(i)))
```

It appears that we have to decide between if the result was random or calculated, 32 times.

## best randomness metric

I tested multiple different metrics for determining randomness, using the column vectors to compare between output bits of the same location rather than to compare laterally.

**Note that the data here only applies to `rand_circuit` outputs. Different patterns produced by other programs may be favoured by different methods and threshold values.**

- `method` $M(X)$, a scoring function from $0$ to $1$ representing how random the data is.
- `shift` Whether to use `1 << i` instead of `i` as input to the circuit.
- `T` A threshold value to determine if its random.
- `Accuracy` Calculated as $\frac{1}{N}\sum^N_{0}{M(X)<T\leq M(R)}$, $R$ is random data.

![optimization_results.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733636133/2024/12/1583457bab9d0c4d1a5d9809812b49e7.png)

*The peaks have high fluctuation due to my optimization algorithm searching those places at higher frequency.*

It is interesting how some of the results are rectangular waves, while some others are normal/sine waves.
### winner / np_entropy_mean

And below is the exact implementation for the best metric `np_entropy_mean`.

It's just a simple column entropy metric.

```python
def binary_entropy(binary_list):
    p = sum(binary_list) / len(binary_list)
    if p == 0 or p == 1:
        return 0
    return -p * math.log2(p) - (1 - p) * math.log2(1 - p)

def np_entropy_mean(ls):
    return np.mean([binary_entropy(list(x)) for x in zip(*ls)])

THRESHOLD = 0.5668030229817654
SHIFT = False
```

## solve

![hist.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733638870/2024/12/dfde6a03eb32c15461656f9c7a1afdc4.png)

Even though the accuracy suggest that $P(\text{Success})=0.9558^{32}=23.54\%$, in practice it appears to have an overall success rate of $75.8\%$, and a true accuracy of $99.14\%$. This may suggest that my accuracy metric wasn't that accurate, but whatever it works.

```python
from matplotlib import pyplot as plt
import numpy as np
from pwn import *
from tqdm import tqdm
from concurrent.futures import ThreadPoolExecutor

THRES = 0.5668030229817654
B = 12
FLAG = ""

def binary_entropy(binary_list):
    p = sum(binary_list) / len(binary_list)
    if p == 0 or p == 1:
        return 0
    return -p * math.log2(p) - (1 - p) * math.log2(1 - p)

def sanity_judgement(ls):
    return np.mean([binary_entropy(list(x)) for x in zip(*ls)])

def solve():
    global FLAG
    context.log_level = "error"
    conn = remote("chall.polygl0ts.ch", 9068)
    # conn = remote("localhost", 5000)
    conn.sendlineafter(b"?", b"3")
    for i in range(32):
        results = []
        for j in range(8):
            conn.sendlineafter(b"test input\n", b"2")
            conn.sendlineafter(b"input: ", str(j).encode())
            conn.recvuntil(b"res = ")
            res = int(conn.recvline().strip())
            results.append(list(map(int, bin(res)[2:].zfill(B))))
        score = sanity_judgement(results)
        conn.sendlineafter(b"test input\n", b"1")
        conn.sendlineafter(b"bit: ", b"0" if score > THRES else b"1")
        try:
            conn.recvline(timeout=0.5)
        except:
            conn.close()
            return i
    if FLAG == "":
        FLAG = conn.recvall().decode().strip()
        print(FLAG)
    conn.close()
    return 32

with ThreadPoolExecutor() as executor:
    results = np.array(list(tqdm(executor.map(lambda x: solve(), range(1000)), total=1000)))
accuracy = np.mean(results == 32)
print(f"Success Rate: {accuracy}")
average_rounds = np.mean(results)
print(f"Average Rounds: {average_rounds}")
np.save("results.npy", results)
plt.style.use('dark_background')
plt.title("Histogram of Stopping Round")
plt.hist(results, bins=range(33))
plt.xlabel("Rounds")
plt.ylabel("Frequency")
plt.savefig("hist.png")
plt.show()
```

```flag
EPFL{r4nd0m_c1rcu1t5_4r3_n0_g00d_rngs??}
```