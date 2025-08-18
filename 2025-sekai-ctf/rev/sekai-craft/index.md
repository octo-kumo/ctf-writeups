---
ai_date: 2025-08-17 20:46:59
ai_summary: Found repeating code with DES-like structure, manual decoding revealed
  flag
ai_tags:
- des
- pattern-recognition
- manual-decoding
created: 2025-08-17T10:48
points: 187
solves: 35
title: sekai-craft
updated: 2025-08-17T20:56
---

I first made a small debugger? that just let me visualize the values in `bits` right before the final comparison, and added a clock to run the function on loop.

What I found was that

- the white bits (`cipherX_XX`) don't change.
- the first half of the program (before the first return) only uses the first half of the input levers
- there is no clear correlation between lever position and final value of `vX_XX`.
- 64 levers -> 64 `vX_XX` values.

![2025-08-17_11.17.30.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1755443927/20250817111847070.png/c83669780a0f8c391d4e60b9ff171c9f.png)

After checking the `mcfunction` file, I can tell that the operation done on the first half of the bits are the same as the ones done on the second half of the bits.

```
v0_0 bit = lever64 bit
v1_0 bit = lever96 bit
v0_1 bit = lever65 bit
v1_1 bit = lever97 bit
v0_2 bit = lever66 bit
v1_2 bit = lever98 bit
v0_3 bit = lever67 bit
v1_3 bit = lever99 bit
v0_4 bit = lever68 bit
v1_4 bit = lever100 bit
v0_5 bit = lever69 bit
v1_5 bit = lever101 bit
v0_6 bit = lever70 bit
v1_6 bit = lever102 bit
v0_7 bit = lever71 bit
v1_7 bit = lever103 bit
v0_8 bit = lever72 bit
v1_8 bit = lever104 bit
v0_9 bit = lever73 bit
v1_9 bit = lever105 bit
v0_10 bit = lever74 bit
v1_10 bit = lever106 bit
v0_11 bit = lever75 bit
v1_11 bit = lever107 bit
v0_12 bit = lever76 bit
v1_12 bit = lever108 bit
v0_13 bit = lever77 bit
v1_13 bit = lever109 bit
v0_14 bit = lever78 bit
v1_14 bit = lever110 bit
v0_15 bit = lever79 bit
v1_15 bit = lever111 bit
v0_16 bit = lever80 bit
v1_16 bit = lever112 bit
v0_17 bit = lever81 bit
v1_17 bit = lever113 bit
v0_18 bit = lever82 bit
v1_18 bit = lever114 bit
v0_19 bit = lever83 bit
v1_19 bit = lever115 bit
v0_20 bit = lever84 bit
v1_20 bit = lever116 bit
v0_21 bit = lever85 bit
v1_21 bit = lever117 bit
v0_22 bit = lever86 bit
v1_22 bit = lever118 bit
v0_23 bit = lever87 bit
v1_23 bit = lever119 bit
v0_24 bit = lever88 bit
v1_24 bit = lever120 bit
v0_25 bit = lever89 bit
v1_25 bit = lever121 bit
v0_26 bit = lever90 bit
v1_26 bit = lever122 bit
v0_27 bit = lever91 bit
v1_27 bit = lever123 bit
v0_28 bit = lever92 bit
v1_28 bit = lever124 bit
v0_29 bit = lever93 bit
v1_29 bit = lever125 bit
v0_30 bit = lever94 bit
v1_30 bit = lever126 bit
v0_31 bit = lever95 bit
v1_31 bit = lever127 bit
```

After slicing out the operations done on single bits.

```sh
❯ cat mvm.mcfunction | grep v0_0 > v0_0.txt
❯ cat mvm.mcfunction | grep v1_0 > v1_0.txt
❯ cat mvm.mcfunction | grep v0_16 > v0_16.txt
❯ cat mvm.mcfunction | grep v1_16 > v1_16.txt
```

The LLM confidently told me that this is running DES.

```
Why It's Definitely DES (Not RSA or AES) 

Let me definitively rule out other possibilities: 
    Not RSA: 
        RSA would show modular exponentiation patterns
        Would have much larger bit sizes (typically 1024+ bits)
        No Feistel structure
        No S-box implementations
    Not AES: 
        AES uses 128-bit blocks (not 64-bit)
        AES has different S-box structure (8-bit to 8-bit)
        AES uses MixColumns operation (matrix multiplication)
        Bit slicing for AES follows a different pattern
    Not Montgomery Multiplication: 
        Montgomery would show specific reduction steps
        Would have a different bit propagation pattern
        No S-box implementations
    DES Confirmation: 
        64-bit block size (perfect match)
        16 rounds (matches the 32 operation repetitions)
        Feistel structure (v0/v1 pairing)
        8 S-boxes (visible in the temporary variable patterns)
        48-bit round keys (visible in tmpT/tmpU variables)
        Exact bit transposition pattern matching DES specifications
```

But DES doesn't use 64 bit keys lol.

Welp, guess I still have to do it myself.

## The repeating code

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1755462779/20250817163259002.png/7692ef2bdfc99a3dfb59ea92047a0226.png)

This big chunk of code is repeated a lot of times.

```python
# parse the code
values = {}
def run_code(line):
    global values
    if '=' in line:
        var, expr = line.split('=')
        var = var.strip()
        expr = expr.strip()
        expr = expr.isdigit() and int(expr) or values.get(expr, 0)
        lc = var[-1]
        if lc in ['+', '-', '*', '/', '%']:
            # This is an operation
            op = lc
            var = var[:-1].strip()
            if op == '+':
                values[var] = values.get(var, 0) + expr
            elif op == '-':
                values[var] = values.get(var, 0) - expr
            elif op == '*':
                values[var] = values.get(var, 0) * expr
            elif op == '/':
                values[var] = values.get(var, 0) // expr
            elif op == '%':
                if expr == 0:
                    print(f"Division by zero in line: {line.strip()}")
                    return
                values[var] = values.get(var, 0) % expr
        else:
            values[var] = expr
    else:
        print(f"Unrecognized line: {line.strip()}")
def save_value(name, value, bits=32):
    global values
    for i in range(bits):
        key = f"{name}_{i}"
        values[key] = (value >> i) & 1
def read_value(name, bits=32):
    global values
    value = 0
    for i in range(bits):
        key = f"{name}_{i}"
        if key in values:
            value |= (values.get(key, 0) << i)
    return value

values = {}
save_value('sum', 123)
save_value('delta', 5928)
for line in code.strip().split('\n'):
    run_code(line)

print("Final values:")
print(f"sum = {read_value('sum')}")
print(f"delta = {read_value('delta')}")
```

After playing around with it I can guess it is just adding two numbers.

Effectively it is doing:

```python
sum += delta
```

---

Turns out this is a pattern, a lot of big chunks are repeating 64 times.

So I added the following to help me find boundaries.

This sort of tells me if the piece of logic has dependencies, which helps me isolate logic blocks.

```python
if expr not in values and not expr.isdigit():
	print("[warning] Uninitialized variable:", expr)
```

### The super long piece
One of them is 1090 lines long!
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1755464104/20250817165503950.png/3a1370e88a4476df99d24694e30688e4.png)

I couldn't tell straight away so I plotted it out.

```python
import matplotlib.pyplot as plt

results = []

for _ in range(1000):
    values = {}
    v0 = random.randint(0, 1000)
    v1 = random.randint(0, 1000)
    kSel2 = 1

    save_value('v0', v0)
    save_value('v1', v1)
    save_value('kSel2', kSel2)
    for line in code.strip().split('\n'):
        run_code(line)

    final_v1 = read_value('v1')
    results.append((v0, v1, final_v1))

results = np.array(results)
fig = plt.figure()
ax = fig.add_subplot(111, projection='3d')
ax.scatter(results[:], results[:], results[:], s=2)
ax.set_xlabel('v0')
ax.set_ylabel('v1 (initial)')
ax.set_zlabel('v1 (final)')
plt.show()
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1755464703/20250817170503591.png/75fe3da7379779966eecb3ef54a53d12.png)

Seems to be linear.

So $A\times v_{0}+B\times v_{1}+C\times v_{1_{f}}=D$, or $v_{1_{f}}=a\times v_{0}+b\times v_{1}+c$

```python
X = results[:, :2]
y = results[:, 2]
X_design = np.hstack([X, np.ones((X.shape[0], 1))])
coeffs, residuals, rank, s = np.linalg.lstsq(X_design, y, rcond=None)
a, b, c = coeffs
print("final_v1 = {:g}*v0 + {:g}*v1 + {:g}".format(a,b,c))
# final_v1 = 17.007146618485226*v0 + 1.000055304352842*v1 + 3.708279205995008
```

The final value doesn't seem to be, decimal?

Let's solve it together with `kSel`

```python
final_v1 = 16.98816041725378*v0 + 1.0036137095375945*v1 + 0.022388593218162778*kSel2 + 8.55173025397057
R^2 = 0.9929852163025292
```

Hrm, numbers aren't very good. Welp.

### Manual work
... And it's not math, its binary logic.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1755468999/20250817181639263.png/7abcf4eb548e34f4c402c3570d03c8f0.png)

After some manual decoding I finally got it.

### The extra switches

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1755469113/20250817181833734.png/78f2d1350858b533c38d84753d62ac0e.png)

## Deobfuscated

```
$zero=0
$two=2
delta=01011011000110011111011101010000;
lever=? i128 // unknown 128 bit value
k0=01011011000111000010111011111010
k1=11011111000001100101111110001111
k2=10011100010001000011100100010100
k3=10011101001101000010000000010001
cipher0=11111111001010111000010000001000
cipher1=10110101011101001101010011000101
cipher2=01111010101100011100001100100000
cipher3=11010010101110100110010110101000
v0=0 i32
v1=0 i32
sum=0 i32
v0=lever & 0xFFFFFFFF       // ls 32 bit
v1=(lever>>32) 0xFFFFFFFF // 32-64 bit
sum+=delta
$i0=sum_0
$i1=sum_1
kSel=$i0?($i1?k3:k1):($i1?k2:k0)
v0+=((v1<<4)^(v1>>5)+v1)^kSel

$j0=sum_11
$j1=sum_12
kSel2=$j0?($j1?k3:k1):($j1?k2:k0)
v1+=((v0<<4)^(v0>>5)+v0)^kSel2

sum+=delta
$i0=sum_0
$i1=sum_1
kSel=$i0?($i1?k3:k1):($i1?k2:k0)
v0+=((v1<<4)^(v1>>5)+v1)^kSel
...

$j0=sum_11
$j1=sum_12
kSel2=$j0?($j1?k3:k1):($j1?k2:k0)
v1+=((v0<<4)^(v0>>5)+v0)^kSel2

$ok=(v0==cipher0)&&(v1==cipher1)
```

I'm almost done!

Both halves are just repeating the following 32 times.

```
sum+=delta
$i0=sum_0
$i1=sum_1
kSel=$i0?($i1?k3:k1):($i1?k2:k0)
v0+=((v1<<4)^(v1>>5)+v1)^kSel
$j0=sum_11
$j1=sum_12
kSel2=$j0?($j1?k3:k1):($j1?k2:k0)
v1+=((v0<<4)^(v0>>5)+v0)^kSel2
```

Or I could write it as:

```
v0 = <unknown>
v1 = <unknown>

sum=0
delta=0b01011011000110011111011101010000
for _ in range(32):
  sum+=delta
  $i0=sum&1
  $i1=sum&2
  kSel=$i0?($i1?k3:k1):($i1?k2:k0)
  v0+=((v1<<4)^(v1>>5)+v1)^kSel
  $j0=(sum >> 11) & 1
  $j1=(sum >> 12) & 1
  kSel2=$j0?($j1?k3:k1):($j1?k2:k0)
  v1+=((v0<<4)^(v0>>5)+v0)^kSel2 

$ok=(v0==cipher0)&&(v1==cipher1) # has to be true
```

All numbers are 32 bit unsigned integers, and we have to find `v0` and `v1`.

LLM identified this as a 32 round Tiny Encryption Algorithm.

## Decoding
I got stuck for a very long time because on the first solve, my endianness was flipped.

I opened a ticket and the author said my numbers were off.

I copied the bits directly in the order in the file, so it was `delta_0delta1...`.

I quickly flipped the bits hoping to get the flag and tried to verify it with my VM, but it failed to pass the test.

On hindsight I should've saw the `)^:d00gt4rc14k3s` string I got out of the decoding and spot the flag because its all plain text, but I trusted my VM too much.

I then spent close to an hour, right before the end of the CTF, trying to debug my logic, not my VM.

Until I added `#break` and `#print` statements to my VM to see the values of variables as the execution happened, did I realise that I had a typo in the length to extract.

> This is after rounds of regex replacement on the `mcfunction` file.

```
if($j0==0&&$j1==0)kSel2_0=k0_0
-> `($j0` ...
```

The variable name extract would have an extra `(` and cause the VM to misbehave, messing up the value of `kSel2` on every iteration.

When I finally realize it was my VM at fault I was once again confused as to why the flag I got is not correct, until I looked at it again and realize that there isn't any flag format, and I just need to flip it.

`)^:d00gt4rc14k3s -> s3k41cr4tg00d:^)`

```flag
SEKAI{s3k41cr4tg00d:^)}
```

## Solve Script

```python [snippet.py]
from test import reset_values, run_code, save_value, read_value, dump_values, code,values
# literally had the flag the entire time i was debugging this shit
save_value('lever', 0x295e3a643030677434726331346b3373, 128)

for line in code.strip().split('\n'):
    if line == '#break':
        dump_values(['v0','v1','sum','kSel','kSel2'])
    if line.startswith('#print'):
        # #print j0=($j0) j1=($j1)
        format = line[6:].strip()
        lk = r"\([^)]+\)"
        import re
        def repl(m):
            var = m.group(0)[1:-1]
            if values.get(var) is None:
                return f"{read_value(var):08x}"
            return f"{values.get(var):08x}"
        s = re.sub(lk, repl, format)
        print(s)
    run_code(line)
dump_values(['v0','v1','sum','kSel','kSel2'])
```

```python [solve.py]
from Crypto.Util.number import long_to_bytes
def uint32(x):
    return x & 0xFFFFFFFF
def tea_decrypt(cipher0, cipher1, k):
    v0 = uint32(cipher0)
    v1 = uint32(cipher1)
    sum_ = uint32((delta * 32) & 0xFFFFFFFF)
    for _ in range(32):
        ksel2 = k[(sum_ >> 11) & 3]
        t = (( (v0 << 4) ^ (v0 >> 5) ) + v0) & 0xFFFFFFFF
        v1 = uint32(v1 - (t ^ ksel2))
        ksel = k[ sum_ & 3 ]
        t = (( (v1 << 4) ^ (v1 >> 5) ) + v1) & 0xFFFFFFFF
        v0 = uint32(v0 - (t ^ ksel))
        sum_ = uint32(sum_ - delta)
    return v0, v1
def tea_encrypt(v0, v1, k):
    v0 = uint32(v0)
    v1 = uint32(v1)
    sum_ = 0
    for _ in range(32):
        sum_ = uint32(sum_ + delta)
        ksel = k[sum_ & 3]
        t = uint32(((v1 << 4) ^ (v1 >> 5)) + v1)
        v0 = uint32(v0 + (t ^ ksel))
        ksel2 = k[(sum_ >> 11) & 3]
        t = uint32(((v0 << 4) ^ (v0 >> 5)) + v0)
        v1 = uint32(v1 + (t ^ ksel2))
        print(f"v0={v0:08x} v1={v1:08x} sum={sum_:08x} kSel={ksel:08x} kSel2={ksel2:08x}")
    return v0, v1

cipher0=270652671
cipher1=2737516205
cipher2=79924574
cipher3=363224395
delta=183474394
k0=1601452250
k1=4059717883
k2=681321017
k3=2281974969

print(f"k0={k0:08x} k1={k1:08x} k2={k2:08x} k3={k3:08x}")

A,B=tea_decrypt(cipher0,cipher1,[k0,k1,k2,k3])
C,D=tea_decrypt(cipher2,cipher3,[k0,k1,k2,k3])
print(f"A: {A:08x}, B: {B:08x}, C: {C:08x}, D: {D:08x}")
_v0, _v1 = tea_encrypt(A, B, [k0, k1, k2, k3])
_v2, _v3 = tea_encrypt(C, D, [k0, k1, k2, k3])
print(f"A: {A:08x}, B: {B:08x}, C: {C:08x}, D: {D:08x}")
flag = D.to_bytes(4, 'big') + C.to_bytes(4, 'big') + B.to_bytes(4, 'big') + A.to_bytes(4, 'big')
flag = int.from_bytes(flag, 'big')
print(f"Flag: {flag:032x}")
print(f"SEKAI{{{long_to_bytes(flag)[::-1].decode()}}}")
```

```python [test.py]
from Crypto.Util.number import long_to_bytes
with open('all.mcfunction', 'r') as f:
    code = f.read()
values = {}
def reset_values():
    global values
    values = {}
def run_code(line):
    global values
    if not line.strip() or line.startswith('#'):
        return True
    if line.startswith('if'):
        # if($i0==0&&$i1==0)kSel_0=k0_0
        condition = line[3:line.index(')')].strip()
        eq1, eq2 = condition.split('&&')
        var11, var12 = eq1.split('==')
        # print(var11, var12)
        var21, var22 = eq2.split('==')
        # print(var21, var22)
        cond = (values.get(var11.strip(), 0) == int(var12.strip())) and (values.get(var21.strip(), 0) == int(var22.strip()))
        if cond:
            var, expr = line[line.index(')') + 1:].split('=')
            var = var.strip()
            expr = expr.strip()
            if expr not in values and not expr.isdigit():
                print("[warning] Uninitialized variable:", expr)
            expr = expr.isdigit() and int(expr) or values.get(expr, 0)
            values[var] = expr
    elif '==' in line:
        # v1_31==cipher3_31
        var1, var2 = line.split('==')
        var1 = var1.strip()
        var2 = var2.strip()
        if var1 in values and var2 in values:
            if values[var1] != values[var2]:
                # print(f"[error] Values do not match: {var1}={values[var1]}, {var2}={values[var2]}")
                return False
        elif var1 in values:
            print(f"[warning] {var2} not initialized, using value from {var1}: {values[var1]}")
            values[var2] = values[var1]
        elif var2 in values:
            print(f"[warning] {var1} not initialized, using value from {var2}: {values[var2]}")
            values[var1] = values[var2]
    elif '=' in line:
        var, expr = line.split('=')
        var = var.strip()
        expr = expr.strip()
        if expr not in values and not expr.isdigit():
            print("[warning] Uninitialized variable:", expr)
        expr = expr.isdigit() and int(expr) or values.get(expr, 0)
        lc = var[-1]
        if lc in ['+', '-', '*', '/', '%', '^']:
            op = lc
            var = var[:-1].strip()
            if op == '+':
                values[var] = values.get(var, 0) + expr
            elif op == '-':
                values[var] = values.get(var, 0) - expr
            elif op == '*':
                values[var] = values.get(var, 0) * expr
            elif op == '/':
                values[var] = values.get(var, 0) // expr
            elif op == '^':
                values[var] = values.get(var, 0) ^ expr
            elif op == '%':
                if expr == 0:
                    print(f"Division by zero in line: {line.strip()}")
                    return
                values[var] = values.get(var, 0) % expr
        else:
            values[var] = expr
    else:
        print(f"Unrecognized line: {line.strip()}")
    return True

def save_value(name, value, bits=32):
    global values
    for i in range(bits):
        key = f"{name}_{i}"
        values[key] = (value >> i) & 1
def read_value(name, bits=32):
    global values
    value = 0
    for i in range(bits):
        key = f"{name}_{i}"
        if key in values:
            value |= (values.get(key, 0) << i)
            # values.pop(key, None)
    return value
def all_names():
    global values
    return set(name[:name.rfind('_')] for name in values.keys())
def dump_values(names=None):
    global values
    if names is None:
        names = all_names()
    for name in names:
        print(f"{name}={read_value(name):08x}", end=' ')
    print()
def test(flag):
    global values
    values = {}
    print(long_to_bytes(flag))
    save_value('lever',flag, 128)
    for line in code.strip().split('\n'):
        if not run_code(line):
            return False
    print("Hooray!! Found flag!", flag)
    return True
```

```flag
SEKAI{s3k41cr4tg00d:^)}
```