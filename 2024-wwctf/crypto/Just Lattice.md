---
ai_date: 2025-04-27 05:24:17
ai_summary: LWE encryption scheme exploited for key recovery using brute force search and known plaintexts.
ai_tags:
  - lwe
  - brute-force
  - probabilistic-key-recovery
created: 2024-11-30T15:25
points: 351
solves: 32
updated: 2025-07-14T09:46
---

The Learning with Errors (LWE) encryption scheme is a lattice-based cryptographic system with the following key mathematical components:

## Parameters

- Modulus: $q \in \mathbb{Z}$
- Dimension: $n \in \mathbb{N}$
- Number of samples: $N \in \mathbb{N}$
- Error distribution standard deviation: $\sigma > 0$

## Key Generation

1. Secret Vector $\vec{s}$: $\vec{s} = \left[1, t_1, t_2, \ldots, t_n\right]$ where $t_i \in \left[0, \frac{q}{2}\right)$
2. Public Matrix $A \in \mathbb{Z}_q^{N \times n}$: $A_{i,j} \sim \text{Uniform}\left(0, \frac{q}{2}\right)$
3. Error Vector $\vec{e} \in \mathbb{Z}_q^N$: $e_i \sim \mathcal{D}_{\sigma}$ (Discrete Gaussian Distribution)
4. Public Parameter $P$: $P = \left[b \mid -A\right]$ where $b = \left(A \vec{t} + \vec{e}\right) \mod q$

## Encryption Process

1. Message Encoding: $M_{\text{encoded}} = \left\lfloor\frac{q}{2}\right\rfloor \cdot M$
2. Ciphertext Computation: $C = \left(P^T \vec{r} + M_{\text{encoded}}\right) \mod q$ where $\vec{r} \in {0,1}^N$

## Decryption Process

$\text{Decrypt}(C, \vec{s}) = \left\lfloor\frac{2}{q} \cdot \vec{s}^T C\right\rfloor \mod 2$

### Mathematical Derivation of Decryption

The core decryption step can be mathematically expressed as:

$\vec{s}^T C \approx \left\lfloor\frac{q}{2}\right\rfloor M + \text{small noise}$

Expanded form: $\vec{s}^T C = \vec{s}^T \left(P^T \vec{r} + M_{\text{encoded}}\right)$

### Brute Force Attack Complexity

The attack complexity is bounded by: $\mathcal{O}\left(q^n\right)$

With success probability depending on: $P\left(\text{correct recovery}\right) = f\left(q, n, \sigma\right)$

### Key Security Properties

1. Computational Hardness: $\text{LWE}_{\vec{s},\chi} \triangleq \text{Distinguish}\left(A, A\vec{s} + \vec{e}\right)$
2. Quantum Resistance: $\text{Solving LWE is believed to be } \mathcal{NP}\text{-hard}$

### Noise Analysis

The decryption noise follows a distribution: $\eta \sim \mathcal{N}\left(0, \sigma^2\right)$

Decryption succeeds when $|\eta| \ll \frac{q}{4}$

### Probabilistic Key Recovery

Let $\mathcal{H}$ be the hypothesis space of possible secret vectors:

$$
\hat{\vec{s}} = \arg\max_{\vec{s} \in \mathcal{H}} \left|\left\{(M_i, C_i) : \text{Decrypt}(C_i, \vec{s}) = M_i\right\}\right|
$$

## Solve

```python
from tqdm import tqdm
import numpy as np

P = [...]
C = [...]


P = np.array(P)
C = np.array(C)
q = 127
n = 3


def enc(P, M, q):
    N = P.shape[0]
    n = len(M)
    r = np.random.randint(0, 2, (n, N))
    Z = np.zeros((n, P.shape[1]), dtype=np.int32)
    Z[:, 0] = 1
    C = np.zeros((n, P.shape[1]), dtype=np.int32)
    for i in range(n):
        C[i] = (np.dot(P.T, r[i]) + (np.floor(q/2) * Z[i] * M[i])) % q
    return C


def dec(C, s, q):
    M = np.zeros(len(C), dtype=np.int32)
    for i in range(len(C)):
        M[i] = round((np.dot(C[i], s) % q) * (2/q)) % 2
    return M


def crack_n1_lwe(P, q, num_samples=200):
    known_messages = np.random.randint(0, 2, num_samples)
    ciphertexts = enc(P, known_messages, q)
    best_success = 0
    best_ts = []
    for potential_ts in tqdm(range(q**n)):
        potential_ts = np.unravel_index(potential_ts, (q,) * n)
        potential_s = np.concat((np.array([1]), np.array(potential_ts)))
        success_count = 0
        decrypted = dec(ciphertexts, potential_s, q)
        success_count = np.sum(known_messages == decrypted)
        if success_count > best_success:
            best_success = success_count
            best_ts = potential_ts
        if success_count == num_samples:
            break
    recovered_s = np.concat((np.array([1]), np.array(best_ts)))
    success_rate = best_success / num_samples
    return recovered_s, success_rate


print(f"{q=}")
recovered_s, success_rate = crack_n1_lwe(P, q)
print(recovered_s, success_rate)
M = dec(C, recovered_s, q)


def unprep(s):
    s = ''.join([str(b) for b in s])
    return ''.join([chr(int(s[i:i+8], 2)) for i in range(0, len(s), 8)])


print(unprep(M))
```

```flag
wwf{1f_y0u_5qu33z3_17_h4rd_3n0u6h_ju1c3_w1ll_c0m3_0u7}
```