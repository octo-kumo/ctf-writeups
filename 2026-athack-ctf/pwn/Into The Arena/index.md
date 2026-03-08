---
created: 2026-03-07T16:44
updated: 2026-03-08T12:37
---

The challenge hides a flag in a random location within a `0x0FFFFFFF` arena each round and you can get 32 bytes of data from that arena.

Obviously brute force is not possible, so I was digging very hard on how to leak addresses and stuff...

But then this just caught my attention.

```c
state = rand();
srand(state << 0xF);
```

What if it cycles?

I wrote a script to check it and indeed, it does, even better, a lot of them are in the same cycle `185697372`, which has length 19.

The solve script is hence simple, just go over the known precomputed cycles in order of occurrence, and just try the first position N times (equal to cycle length).

```flag
ATHACKCTF{youtu.be/-f3_odnbDig}
```

## solve

```python
import random
from tqdm import tqdm
from pwn import *
# seed, occurrence, cycle_length
cycles = [
    [185697372, 812, 19],
    [1601770915, 112, 52],
    [2082983190, 22, 52],
    [1580092370, 16, 52],
    [1504306880, 8, 52],
    [833330545, 5, 52],
    [589939653, 5, 52],
    [23042437, 2, 19],
    [260777354, 2, 52],
    [1382447743, 1, 52],
    [606697673, 1, 54],
    [823013598, 1, 54],
    [493538984, 1, 52],
    [1901442168, 1, 54],
    [877185650, 1, 52],
    [2039887290, 1, 54],
    [303457943, 1, 54],
    [919658464, 1, 54],
    [1813066995, 1, 52],
    [1984191101, 1, 54],
    [1663177220, 1, 54],
    [1524071724, 1, 52],
    [626576519, 1, 52],
    [1475782745, 1, 52],
    [736624863, 1, 52],
]

# define ARENA_SIZE 0x0FFFFFFF
# define SECRET_SIZE 32
ARENA_SIZE = 0x0FFFFFFF
SECRET_SIZE = 32
# p = process('./chall')
p = remote('127.0.0.1', 32917)

p.recvuntil(b'Entrance of the arena: ')
arena = int(p.recvline().strip(), 16)
print(f"Received arena address: {hex(arena)}")
existing = set()
total = sum(cycle[2] for cycle in cycles)  # total number of checks
bar = tqdm(desc="Searching for the flag", total=total)

for checking_cycle in range(len(cycles)):
    r_res = cycles[checking_cycle][0]  # seed
    bar.set_description(
        f"Cycle {checking_cycle + 1}/{len(cycles)} (occurrence: {cycles[checking_cycle][1]}, cycle length: {cycles[checking_cycle][2]})")
    for _ in range(cycles[checking_cycle][2]):  # length of the cycle
        offset = r_res % (ARENA_SIZE - SECRET_SIZE - 1)
        # send 0xaddress of the flag (in hex)
        addr = arena + offset
        p.sendlineafter(b'> ', hex(addr).encode())
        res = p.recvuntil(b'Where do you wanna search? ')
        bar.update(1)
        if res in existing:
            continue
        existing.add(res)
        print(res)

'''
❯ uv run solve.py
[+] Opening connection to 127.0.0.1 on port 32917: Done
Received arena address: 0x7c594a297010
Cycle 1/25 (occurrence: 812, cycle length: 19):   0%|                                                                              | 1/1250 [00:06<2:05:47,  6.04s/it]
b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00The Retiariua hides somewhere in the arena...\nDone!\n\x00Where do you wanna search? '
Cycle 2/25 (occurrence: 112, cycle length: 52):   5%|███▌                                                                         | 58/1250 [02:58<1:00:18,  3.04s/it]
b'ATHACKCTF{youtu.be/-f3_odnbDig}\x00The Retiariua hides somewhere in the arena...\nDone!\n\x00Where do you wanna search? '
Cycle 3/25 (occurrence: 22, cycle length: 52):   7%|█████▉                                                                          | 92/1250 [04:41<58:22,  3.02s/it]
'''
```

```c [testcycles.c]
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <unistd.h>
#include <fcntl.h>

#define RUNS 1000

uint32_t next_state(uint32_t state)
{
    srand(state << 15);
    return rand();
}

static uint32_t urandom32(void)
{
    uint32_t x;
    int fd = open("/dev/urandom", O_RDONLY);
    read(fd, &x, sizeof(x));
    close(fd);
    return x;
}

int find_state(uint32_t *arr, int count, uint32_t val)
{
    for (int i = 0; i < count; i++) {
        if (arr[i] == val)
            return i;
    }
    return -1;
}

void verify_cycle(uint32_t seed, uint32_t length)
{
    uint32_t s = seed;

    for (uint32_t i = 0; i < length; i++)
        s = next_state(s);

    if (s != seed)
        printf("Verification failed for seed %u (length %u)\n", seed, length);
}

void sort_results(uint32_t *states, int *occ, uint32_t *lengths, int n)
{
    for (int i = 0; i < n - 1; i++) {
        for (int j = i + 1; j < n; j++) {
            if (occ[j] > occ[i]) {
                int tmp_occ = occ[i];
                occ[i] = occ[j];
                occ[j] = tmp_occ;

                uint32_t tmp_state = states[i];
                states[i] = states[j];
                states[j] = tmp_state;

                uint32_t tmp_len = lengths[i];
                lengths[i] = lengths[j];
                lengths[j] = tmp_len;
            }
        }
    }
}

int main()
{
    uint32_t cycle_states[RUNS];
    uint32_t cycle_lengths[RUNS];
    int occurrences[RUNS];

    int unique_count = 0;

    for (int run = 0; run < RUNS; run++) {

        uint32_t start = urandom32();

        uint32_t tortoise = next_state(start);
        uint32_t hare = next_state(next_state(start));

        while (tortoise != hare) {
            tortoise = next_state(tortoise);
            hare = next_state(next_state(hare));
        }

        tortoise = start;

        while (tortoise != hare) {
            tortoise = next_state(tortoise);
            hare = next_state(hare);
        }

        uint32_t cycle_start = tortoise;

        uint32_t length = 1;
        hare = next_state(tortoise);

        while (hare != tortoise) {
            hare = next_state(hare);
            length++;
        }

        int idx = find_state(cycle_states, unique_count, cycle_start);

        if (idx >= 0) {
            occurrences[idx]++;
        } else {
            cycle_states[unique_count] = cycle_start;
            cycle_lengths[unique_count] = length;
            occurrences[unique_count] = 1;
            unique_count++;
        }
    }

    sort_results(cycle_states, occurrences, cycle_lengths, unique_count);

    printf("Unique cycle start states: %d\n\n", unique_count);

    for (int i = 0; i < unique_count; i++) {

        verify_cycle(cycle_states[i], cycle_lengths[i]);

        printf("[%u, %d, %u],\n",
               cycle_states[i],
               occurrences[i],
               cycle_lengths[i]);
    }

    return 0;
}
```
