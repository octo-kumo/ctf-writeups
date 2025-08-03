---
created: 2025-08-03T13:23
updated: 2025-08-03T15:59
description: AI flag checker (hard)
---

## post mortem

Yay! non-zero solves!

The one thing I messed up was the final score, because the original challenge was not linear layers, it was a CNN which was much smaller in size, and is also more accurate.

However my intended solution of generating a image just by passing a `1050*51` image over and over to get the flag didn't work. I should've just tried to solve it with other algorithms like the other player did, which actually uses the font data, but it was late in the night and also approaching the start of the CTF, so I just made a new model that is fully linear, hopefully making the adversarial attack easier.

But I still messed up the font rendering part, this caused the glyphs to not match the training code, this caused the score to be low (~0.1). I couldn't figure out the solution in time, so I just lowered the required score to pass the password check.

This caused confusion during the CTF as people are able to pass the password check with gibberish flags.

Next time I shouldn't be making challenges last second 😭

## introduction

> This write up is for the hardest version of this challenge, however as you will see they are all just the same challenge.

Let's first try with `upx`.

```bash
$ upx -d chall
```

You might realise that `start` is running `lzma` decompression.

`_BYTE *__fastcall start(_BYTE *a1, __int64 a2, __int64 a3)`

- `a1` - appears to be a pointer to compressed data
- `a2` - likely the size of the compressed data
- `a3` - possibly output buffer or size parameter

So this is `upx` with `lzma`.

> It appears that a lot of people failed to solve the first part due to `upx` version mismatch, interesting.

```bash
$ upx -d --lzma chall
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2024
UPX 4.2.4       Markus Oberhumer, Laszlo Molnar & John Reiser    May 9th 2024

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
  54955711 <-  49500040   90.07%   linux/amd64   chall

Unpacked 1 file.
```

> After the first step we arrive at `chall_easier`, the second binary distributed.

Next, a pass through any LLM might tell you that this is an AI model.

But since this is the author's write up, let's try to solve this manually.

## setup

This part just uses the first command line argument for the flag and check if its length is 50.

```c
if ( a1 <= 1 )
{
  __printf_chk(2, "Usage: %s <password>\n", *a2);
  return 1;
}
v3 = a2[1];
if ( strlen(v3) != 50 )
{
  fwrite("Error: password length must be exactly 50 chars.\n", 1u, 0x31u, stderr);
  return 1;
}
```

Then it setups the random seed, and does a fancy array setup on `v8`, which is a float array at `&unk_349C960 - 4192`.

Notice that `_mm_shuffle_ps((__m128)0xBF800000, (__m128)0xBF800000, 0);` is just making four `-1` floats `[-1.0f, -1.0f, -1.0f, -1.0f]`.

So this is might be some optimised code to set 4 floats at once, and it basically fills `v8` (or `v7`) with `-1`.

- 262 `m128` + 8 bytes = 1048 floats + 2 floats = 1050 floats
- Some unknown iterations on top of that.

```c
v4 = time(0);
srand(v4);
rand();
v5 = (__m128 *)&unk_349C960;
v6 = (__m128)_mm_loadl_epi64((const __m128i *)&xmmword_3465C00);
v7 = (char *)&unk_349C960 - 4192;
v8 = (__m128 *)((char *)&unk_349C960 - 4192);
v9 = _mm_shuffle_ps((__m128)0xBF800000, (__m128)0xBF800000, 0);
do
{
  v10 = v8;
  if ( (((_BYTE)v5 - (_BYTE)v8) & 0x10) == 0 || (v10 = v8 + 1, *v8 = v9, &v8[1] != v5) )
  {
    do
    {
      *v10 = v9;
      v10 += 2;
      v10[-1] = v9;
    }
    while ( v10 != v5 );
  }
  v5 = (__m128 *)((char *)v10 + 4200);
  _mm_storel_ps((double *)v10->m128_u64, v6);
  v8 = (__m128 *)((char *)v8 + 4200);
}
while ( (unsigned __int16 *)((char *)&unk_344B8 + (_QWORD)&unk_349C960) != &v10[262].m128_u16[4] );
v11 = (__m128)_mm_loadl_epi64((const __m128i *)&xmmword_3465C10);
v12 = v3;
```

## char rendering

Now this part, it might take a bit more thinking (or guessing) but its pretty clear with how $1050/21=50$ that this is running a for loop on each char.

> Congrats to @oh_word for finding this out first, I think he is first.

```c
for ( i = 0; i != 1050; i += 21 )
```

It is also checking the flag against a charset, if it fails it will exit.

```c
v15 = strchr("0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!\"#$%&'()*+,./:;<=>?@[\\]^_`{|}~", v14);
v16 = v100;
v6 = (__m128)_mm_loadl_epi64((const __m128i *)&v102);
v11 = v103;
if ( !v15 )
{
  __fprintf_chk(stderr, 2, "Unknown char: %c\n", v14);
  return 1;
}
```

Here `v17` is the index of the char in the charset.

Then it is loading some meta data (`v18` `v19` `v20` `v104`) about the character, so let's extract them.

```c
v17 = (int)((_DWORD)v15
          - (unsigned int)"0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!\"#$%&'()*+,./:;<=>?@[\\]^_`{|}~");
v18 = byte_9E40[v17];
v19 = byte_9DE0[v17];
v20 = dword_9C60[v17];
v21 = 21 - v18;
v104 = byte_9E40[v17];
```

```c
.x3:0000000000009C60 ; unsigned int dword_9C60[96]
.x3:0000000000009C60 dword_9C60      dd 0, 1A9h, 2A3h, 44Ch, 5F5h, 7B7h, 960h, 0B09h, 0CB2h
.x3:0000000000009C60                                         ; DATA XREF: main+142↑o
.x3:0000000000009C84                 dd 0E5Bh, 1004h, 1136h, 12C6h, 13E6h, 1576h, 16A8h, 17BBh
.x3:0000000000009CA4                 dd 194Bh, 1AC2h, 1B26h, 1BE6h, 1D5Dh, 1DC1h, 1F17h, 2025h
.x3:0000000000009CC4                 dd 2157h, 22E7h, 2477h, 252Bh, 2639h, 2733h, 2841h, 2973h
.x3:0000000000009CE4                 dd 2AEDh, 2C1Fh, 2DC8h, 2EFAh, 3107h, 32E2h, 34D6h, 36B1h
.x3:0000000000009D04                 dd 388Ch, 3A4Eh, 3C42h, 3E1Dh, 3E81h, 3FDFh, 41BAh, 434Ah
.x3:0000000000009D24                 dd 4525h, 4700h, 48F4h, 4ACFh, 4CC3h, 4E9Eh, 5092h, 529Fh
.x3:0000000000009D44                 dd 547Ah, 5687h, 5894h, 5AA1h, 5CAEh, 5EA2h, 5F1Fh, 5F79h
.x3:0000000000009D64                 dd 6154h, 6341h, 6535h, 6729h, 674Dh, 686Dh, 698Dh, 69FBh
.x3:0000000000009D84                 dd 6B1Ch, 6B44h, 6B50h, 6C4Ah, 6C92h, 6D05h, 6E26h, 6ED0h
.x3:0000000000009DA4                 dd 6FF1h, 719Ah, 741Ah, 74FAh, 75F4h, 76F4h, 77C4h, 7800h
.x3:0000000000009DC4                 dd 7823h, 7983h, 79E3h, 7B43h, 3 dup(0)
.x3:0000000000009DE0 ; unsigned __int8 byte_9DE0[96]
.x3:0000000000009DE0 byte_9DE0       db 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 12h, 19h, 12h, 19h, 12h, 19h, 19h, 19h, 19h, 20h, 19h, 19h, 12h, 12h, 12h, 19h, 19h, 12h, 12h, 19h, 12h, 12h, 12h, 12h, 19h, 12h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 19h, 9, 19h, 1Dh, 19h, 19h, 9, 20h, 20h, 0Ah, 11h, 8, 3, 19h, 12h, 17h, 11h, 0Ah, 11h, 19h, 20h, 20h, 19h, 20h, 0Dh, 3, 5, 20h, 20h, 20h, 6, 0, 0, 0
.x3:0000000000009DE0                                         ; DATA XREF: main+125↑o
.x3:0000000000009E40 ; unsigned __int8 byte_9E40[96]
.x3:0000000000009E40 byte_9E40       db 11h, 0Ah, 11h, 11h, 12h, 11h, 11h, 11h, 11h, 11h, 11h, 10h, 10h, 10h, 11h, 0Bh, 10h, 0Fh, 4, 6, 0Fh, 4, 13h, 0Fh, 11h, 10h, 10h, 0Ah, 0Fh, 0Ah, 0Fh, 11h, 15h, 11h, 11h, 11h, 15h, 13h, 14h, 13h, 13h, 12h, 14h, 13h, 4, 0Eh, 13h, 10h, 13h, 13h, 14h, 13h, 14h, 13h, 14h, 15h, 13h, 15h, 15h, 15h, 15h, 14h, 5, 0Ah, 13h, 11h, 14h, 14h, 4, 9, 9, 0Bh, 11h, 5, 4, 0Ah, 4, 5, 11h, 11h, 11h, 11h, 14h, 7, 0Ah, 8, 10h, 14h, 7, 0Bh, 3, 0Bh, 12h, 0, 0, 0
```

The next part seems to be adjusting some parameters, halving them after.

```c
if ( 21 - v18 < 0 )
  v21 = 0;
v22 = v21 >> 1;
v23 = 51 - v19;
if ( 51 - v19 < 0 )
  v23 = 0;
v24 = v23 >> 1;
```

---

We could infer that this part is copying the raw pixel values from `byte_20A0`, and the following.

- Column start: `21*k + v22`, where `v22 = max(0, (21 - v18) / 2)` centres the patch in a 21-unit block.
- Row start: `v24`, where `v24 = max(0, (51 - v19) / 2)` centres the patch in a 51-unit block.
- Size: `v18*v19` bytes from `byte_20A0[v20]`.
- Conversion: Bytes (0-255) are scaled by $0.0078431377\approx2/255$ and shifted by $-1.0$, resulting in a range of $[-1, 1]$.

So:

- `byte_9E40`: `v18` (width of char, 0-255).
- `byte_9DE0`: `v19` (height of char, 0-255).
- `dword_9C60`: `v20` (offset into `byte_20A0`).

```c
if ( v19 )
{
  v25 = i + v22;
  if ( v18 )
  {
	v105 = i;
	v106 = v100;
	v26 = &byte_20A0[v20];
	v27 = v25 + 1050LL * v24;
	v28 = v19 + v24;
	v29 = 0;
	v101 = v28;
	v102.m128_i32[0] = v18 - 1;
	v103.m128_u64[0] = 16LL * ((unsigned int)v18 >> 4);
	do
	{
	  if ( v102.m128_i32[0] <= 0xEu )
	  {
		v37 = 0;
		v38 = 0;
	  }
	  else
	  {
		v30 = v104 & 0xF;
		while ( 1 )
		{
		  v31 = &v7[4 * v27];
		  v32 = (const __m128i *)v26;
		  v33 = (const __m128i *)&v26[v103.m128_u64[0]];
		  do
		  {
			v34 = _mm_loadu_si128(v32++);
			v31 += 64;
			v35 = _mm_unpacklo_epi8(v34, (__m128i)0LL);
			*((__m128 *)v31 - 3) = _mm_add_ps(
									 _mm_mul_ps(
									   _mm_cvtepi32_ps(_mm_unpackhi_epi16(v35, (__m128i)0LL)),
									   (__m128)xmmword_3465C10),
									 (__m128)xmmword_3465C00);
			v36 = _mm_unpackhi_epi8(v34, (__m128i)0LL);
			*((__m128 *)v31 - 4) = _mm_add_ps(
									 _mm_mul_ps(
									   _mm_cvtepi32_ps(_mm_unpacklo_epi16(v35, (__m128i)0LL)),
									   (__m128)xmmword_3465C10),
									 (__m128)xmmword_3465C00);
			*((__m128 *)v31 - 1) = _mm_add_ps(
									 _mm_mul_ps(
									   _mm_cvtepi32_ps(_mm_unpackhi_epi16(v36, (__m128i)0LL)),
									   (__m128)xmmword_3465C10),
									 (__m128)xmmword_3465C00);
			*((__m128 *)v31 - 2) = _mm_add_ps(
									 _mm_mul_ps(
									   _mm_cvtepi32_ps(_mm_unpacklo_epi16(v36, (__m128i)0LL)),
									   (__m128)xmmword_3465C10),
									 (__m128)xmmword_3465C00);
		  }
		  while ( v32 != v33 );
		  if ( v30 )
			break;
		  v26 += (unsigned __int8)v18;
		  v29 += v18;
		  ++v24;
		  v27 += 1050;
		  if ( v101 == v24 )
			goto LABEL_33;
		}
		v37 = v18 & 0xF0;
		v38 = v18 & 0xFFFFFFF0;
	  }
	  v39 = v18 - v37;
	  if ( v18 - v37 - 1 <= 6 )
		goto LABEL_58;
	  v40 = _mm_loadl_epi64((const __m128i *)&v26[v37]);
	  v41 = (double *)&v7[4 * v37 + 4 * v27];
	  v42 = _mm_unpacklo_epi8(v40, (__m128i)0LL);
	  _mm_storel_ps(
		v41 + 1,
		_mm_add_ps(
		  _mm_mul_ps(
			_mm_cvtepi32_ps(_mm_move_epi64(_mm_shuffle_epi32(_mm_unpacklo_epi16(v42, (__m128i)0LL), 78))),
			v11),
		  v6));
	  v43 = _mm_unpacklo_epi16(_mm_shuffle_epi32(v42, 78), (__m128i)0LL);
	  _mm_storel_ps(
		v41,
		_mm_add_ps(_mm_mul_ps(_mm_cvtepi32_ps(_mm_move_epi64(_mm_unpacklo_epi16(v42, (__m128i)0LL))), v11), v6));
	  _mm_storel_ps(v41 + 2, _mm_add_ps(_mm_mul_ps(_mm_cvtepi32_ps(_mm_move_epi64(v43)), v11), v6));
	  _mm_storel_ps(
		v41 + 3,
		_mm_add_ps(_mm_mul_ps(_mm_cvtepi32_ps(_mm_move_epi64(_mm_shuffle_epi32(v43, 78))), v11), v6));
	  v38 += v39 & 0xFFFFFFF8;
	  if ( (v39 & 7) != 0 )
	  {
LABEL_58:
		v44 = &byte_20A0[v20];
		v45 = 1050LL * v24;
		*(float *)&v7[4 * v45 + 4 * (int)(v25 + v38)] = (float)((float)byte_20A0[v20 + (int)(v38 + v29)]
															  * 0.0078431377)
													  - 1.0;
		v46 = v38 + 1;
		if ( v18 > (int)(v38 + 1) )
		{
		  v47 = v45 + (int)(v25 + v46);
		  v48 = (float)v44[v29 + v46];
		  v49 = v38 + 2;
		  *(float *)&v7[4 * v47] = (float)(v48 * 0.0078431377) - 1.0;
		  if ( v18 > (int)(v38 + 2) )
		  {
			v50 = v45 + (int)(v25 + v49);
			v51 = (float)v44[v29 + v49];
			v52 = v38 + 3;
			*(float *)&v7[4 * v50] = (float)(v51 * 0.0078431377) - 1.0;
			if ( v18 > (int)(v38 + 3) )
			{
			  v53 = v45 + (int)(v25 + v52);
			  v54 = (float)v44[v29 + v52];
			  v55 = v38 + 4;
			  *(float *)&v7[4 * v53] = (float)(v54 * 0.0078431377) - 1.0;
			  if ( v18 > (int)(v38 + 4) )
			  {
				v56 = v45 + (int)(v25 + v55);
				v57 = (float)v44[v29 + v55];
				v58 = v38 + 5;
				*(float *)&v7[4 * v56] = (float)(v57 * 0.0078431377) - 1.0;
				if ( v18 > (int)(v38 + 5) )
				{
				  v59 = v38 + 6;
				  *(float *)&v7[4 * v45 + 4 * (int)(v25 + v58)] = (float)((float)v44[v29 + v58] * 0.0078431377)
																- 1.0;
				  if ( v18 > v59 )
					*(float *)&v7[4 * v45 + 4 * (int)(v25 + v59)] = (float)((float)v44[v29 + v59] * 0.0078431377)
																  - 1.0;
				}
			  }
			}
		  }
		}
	  }
	  v26 += (unsigned __int8)v18;
	  v29 += v18;
	  ++v24;
	  v27 += 1050;
	}
	while ( v101 != v24 );
LABEL_33:
	i = v105;
	v16 = v106;
  }
}
v12 = v16 + 1;
```

## the neural network

Architecture:

- Input: $1050\times 51$ floats (53550 elements).
- Layer 1: Linear transformation to 256 outputs (weights at `unk_1A400`, biases at `unk_A000`), followed by ReLU.
- Layer 2: Linear transformation to 64 outputs (weights at `unk_A400`, biases at `unk_9F00`), followed by ReLU.
- Layer 3: Linear transformation to 1 output (weights at `xmmword_3465C20` to `3465D10`), no ReLU.

Threshold: Final value > 0.134946639 for "correct".

```c
memcpy(&xmmword_3467440, v7, (size_t)&unk_344B8); // Copy buffer
// First Layer
v62 = (__int128 *)&unk_1A400;
v63 = 0;
do {
    v67 = 0;
    do {
        v68 = _mm_mul_ps(*(__m128 *)((char *)v66 + v62), *(__m128 *)((char *)v66 + v65));
        v66 += 2;
        v67 = _mm_add_ps(v67, v68);
    } while ( v66 != &qword_344B0 );
    v62 = (__int128 *)((char *)v62 + 214200);
    v70 = (__m128i)_mm_add_ps(...);
    *(float *)((char *)xmmword_3467040 + v63) = sum + *(float *)((char *)&unk_A000 + v63);
    v63 += 4;
} while ( v62 != &xmmword_3465C00 );

// ReLU
do {
    v73 = (__m128)xmmword_3467040[v71];
    v74 = _mm_cmplt_ps((__m128)0LL, v73);
    *(__m128 *)&v65[v71 * 16] = _mm_or_ps(_mm_andnot_ps(v74, ...), _mm_and_ps(v73, v74));
    ++v71;
} while ( v71 != 64 );

// Second Layer
v75 = (char *)&unk_A400;
v76 = 0;
do {
    v78 = 0;
    do {
        v79 = _mm_mul_ps(*(__m128 *)&v65[v77], *(__m128 *)&v75[v77]);
        v77 += 16;
        v78 = _mm_add_ps(v78, v79);
    } while ( v77 != 1024 );
    v75 += 1024;
    *(float *)((char *)xmmword_3467040 + v76) = sum + *(float *)((char *)&unk_9F00 + v76);
    v76 += 4;
} while ( v75 != (char *)&unk_1A400 );

// ReLU and Final Layer
v97 = _mm_add_ps(_mm_mul_ps((__m128)xmmword_3467440, (__m128)xmmword_3465CD0), ...); // 16 multiplications and sums
v98 = _mm_add_ps(_mm_movehl_ps(v97, v97), v97);
if ( (float)((float)(_mm_shuffle_ps(v98, v98, 85).m128_f32[0] + v98.m128_f32[0]) - 0.034946639) <= 0.1 )
    puts("Password is incorrect.");
else
    puts("Password is correct!");
```

---

So we've figured out what the binary is doing, it is just a neural network classifier that accepts a flag, renders it, then classifies it.

How do we solve for the flag?

Bruteforcing it will not be possible.

> Post mortem: it might be actually possible, but you will get nonsense.

## solution: adversarial attack

> the author hinted to not trust the font too much.

We can extract the weights, put it in a AI library, and run adversarial attack on it, ignoring the font rendering.

Hopefully it will produce a image that is readable.

### extraction

We first need to extract the data from the binary, use `elftools` for easier extraction.

```python
import elftools.elf.sections
from elftools.elf.elffile import ELFFile

to_extract = [
    ("width", 0x9E40, 94, "uint8"),                    # byte_9E40: Widths for character patches (94 bytes)
    ("height", 0x9DE0, 94, "uint8"),                  # byte_9DE0: Heights for character patches (94 bytes)
    ("offset", 0x9C60, 376, "int32"),                 # dword_9C60: Offsets into byte_20A0 (94 * 4 bytes)
    ("patch_data", 0x20A0, 0x1A400-0x20A0, "uint8"), # byte_20A0: Patch data, just read everything until the next section (0x1A400)
    ("layer1_weights", 0x1A400, 256 * 53550 * 4, "float32"),
    ("layer1_biases", 0xA000, 256 * 4, "float32"),
    ("layer2_weights", 0xA400, 64 * 256 * 4, "float32"),
    ("layer2_biases", 0x9F00, 64 * 4, "float32"),
    ("layer3_weights", 0x3465C20, 64 * 4, "float32")
]
data = {}
with open('/work/chall', 'rb') as f:
    elf = ELFFile(f)

    for name, addr, size, _type in to_extract:
        for sec in elf.iter_sections():
            s = sec['sh_addr']
            e = s + sec['sh_size']
            if s <= addr < e:
                f.seek(sec['sh_offset'] + addr - s)
                data[name] = f.read(size)
                print(f"Extracted {name} at {hex(addr)}: {len(data[name])} bytes")
                break
```

As an sanity check let's first just draw out the `patch_data`.

```python

import numpy as np
from PIL import Image

charset = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!\"#$%&'()*+,./:;<=>?@[\\]^_`{|}~"

widths = np.frombuffer(data["width"], dtype=np.uint8)
heights = np.frombuffer(data["height"], dtype=np.uint8)
offsets = np.frombuffer(data["offset"], dtype=np.int32)
patch_data = np.frombuffer(data["patch_data"], dtype=np.uint8)

num_chars = len(charset)
grid_cols = 10
grid_rows = (num_chars + grid_cols - 1) // grid_cols
max_width = int(widths.max())
max_height = int(heights.max())

cell_width = max_width + 2
cell_height = max_height + 2
image_width = cell_width * grid_cols
image_height = cell_height * grid_rows
image = np.zeros((image_height, image_width), dtype=np.uint8)
for idx, char in enumerate(charset):
    w, h = int(widths[idx]), int(heights[idx])
    offset = offsets[idx]
    patch = patch_data[offset:offset + w * h].reshape(h, w)
    row = idx // grid_cols
    col = idx % grid_cols
    x = col * cell_width + 1
    y = row * cell_height + 1
    x_offset = (max_width - w) // 2
    y_offset = (max_height - h) // 2
    image[y + y_offset:y + y_offset + h, x + x_offset:x + x_offset + w] = patch

img = Image.fromarray(image)
img.save("charset_patches.png")
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1754244764/20250803141244518.png/4b8d2cb2e20912e98453b107db2f68a3.png)

And it is indeed the characters.

### the network

```python
import torch
import torch.nn as nn
import numpy as np
from PIL import Image
import elftools.elf.sections
from elftools.elf.elffile import ELFFile

from tqdm import trange
import matplotlib.pyplot as plt

class FancyChecker(nn.Module):
    def __init__(self):
        super().__init__()
        self.layer1 = nn.Linear(53550, 256)
        self.layer2 = nn.Linear(256, 64)
        self.layer3 = nn.Linear(64, 1)
        self.leaky_relu = nn.LeakyReLU(0.1)

    def forward(self, x):
        print(x.shape)
        x = x.view(x.size(0), -1)
        x = self.leaky_relu(self.layer1(x))
        x = self.leaky_relu(self.layer2(x))
        x = self.layer3(x)
        return x

model = FancyChecker()

model.layer1.weight.data = torch.from_numpy(np.frombuffer(data["layer1_weights"], dtype=np.float32).reshape(256, 53550)).float()
model.layer1.bias.data = torch.from_numpy(np.frombuffer(data["layer1_biases"], dtype=np.float32)).float()
model.layer2.weight.data = torch.from_numpy(np.frombuffer(data["layer2_weights"], dtype=np.float32).reshape(64, 256)).float()
model.layer2.bias.data = torch.from_numpy(np.frombuffer(data["layer2_biases"], dtype=np.float32)).float()
model.layer3.weight.data = torch.from_numpy(np.frombuffer(data["layer3_weights"], dtype=np.float32).reshape(1, 64)).float()
model.layer3.bias.data = torch.from_numpy(np.array([-0.03494664])).float()
```

### the attack

```python

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print(f"Using device: {device}")
model.to(device)
model.eval()

N = 10
alpha = 10000
x = torch.full((1, 1, 51, 1050), -1.0).to(device).detach()
x.requires_grad = True
pbar = trange(N)
images = []
for i in pbar:
    if i % (N//10)==0:
        img = x.detach().cpu().squeeze(0).squeeze(0).numpy()
        images.append((img + 1) / 2 )
    z = model(x)
    pbar.set_postfix({'score': f'{z.item():.4f}'})
    z.backward()
    grad = x.grad
    with torch.no_grad():
        x = x + alpha * grad
        x = torch.clamp(x, -1, 1)
    x.requires_grad = True

x = x.detach()
with torch.no_grad():
    score = model(x).item()

print(f"score after {N} iterations: {score}")

img = np.vstack(images)
img = Image.fromarray((img * 255).astype(np.uint8))
img.save('output.png')
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1754249586/20250803153305592.png/fa1dad266f87d3a36a29a193d1eadd79.png)

```flag
wwf{wait---it-is-a-ai-challenge??-always-has-been}
wwf{wait___it_is_a_ai_challenge??_always_has_been}
```

> I accepted both flags

## other solutions

### Beam Search (@5h1kh4r)

Do a beam search by iteratively scoring rendered candidate flags.

### greedy (@grok)

I tried using grok to solve the challenge after `upx -d --lzma`, it managed to solve everything basically.

> This is the decompiled code of a CTF challenge, it accepts a 50 char flag and checks it. It is a 50MB binary but this is most of the code already, there aren't that much code in the binary. Please read the code carefully, figure out what it is doing, and help me solve for the flag. If there are assets to extract, tell me their names and addresses.

However it did not use pytorch, it just suggested a greedy algorithm, which I did not test, however it might still be interesting to see.

By the looks of it, it seems to just be a hill climb too though.

Also note that grok was fooled by the code optimisations setting 4 floats at once, causing it to think the layers are 4 times smaller than they actually are.

```python
import numpy as np

# Load extracted data
charset = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!\"#$%&'()*+,./:;<=>?@[\\]^_`{|}~"
byte_9E40 = np.fromfile("byte_9E40.bin", dtype=np.uint8)  # [94]
byte_9DE0 = np.fromfile("byte_9DE0.bin", dtype=np.uint8)  # [94]
dword_9C60 = np.fromfile("dword_9C60.bin", dtype=np.int32)  # [94]
byte_20A0 = np.fromfile("byte_20A0.bin", dtype=np.uint8)  # [large]
w1 = np.fromfile("unk_1A400.bin", dtype=np.float32).reshape(64, 1050*51)
b1 = np.fromfile("unk_A000.bin", dtype=np.float32)  # [64]
w2 = np.fromfile("unk_A400.bin", dtype=np.float32).reshape(16, 64)
b2 = np.fromfile("unk_9F00.bin", dtype=np.float32)  # [16]
w3 = np.fromfile("xmmword_3465C20.bin", dtype=np.float32)  # [16]

def forward_pass(flag):
    buffer = np.full((51, 1050), -1.0, dtype=np.float32)
    for k, char in enumerate(flag):
        v17 = charset.index(char)
        v18, v19, v20 = byte_9E40[v17], byte_9DE0[v17], dword_9C60[v17]
        v22 = max(0, (21 - v18) // 2)
        v24 = max(0, (51 - v19) // 2)
        patch = byte_20A0[v20:v20 + v18*v19].reshape(v19, v18)
        patch = patch.astype(np.float32) * 0.0078431377 - 1.0
        col_start = 21 * k + v22
        buffer[v24:v24+v19, col_start:col_start+v18] = patch

    x = buffer.flatten()
    x = np.maximum(w1 @ x + b1, 0)  # Layer 1 + ReLU
    x = np.maximum(w2 @ x + b2, 0)   # Layer 2 + ReLU
    return w3 @ x                    # Final output
flag = list("a" * 50)  # Initial guess
while True:
    improved = False
    for pos in range(50):
        best_char, best_score = flag[pos], -float('inf')
        current = flag.copy()
        for c in charset:
            current[pos] = c
            score = forward_pass("".join(current))
            if score > best_score:
                best_char, best_score = c, score
        if best_char != flag[pos]:
            flag[pos] = best_char
            improved = True
    if not improved or best_score > 0.134946639:
        break

print("Flag:", "".join(flag), "Score:", best_score)
```

### look at the weights (@unknown)

Someone just drew all of layer 1's weights out and apparently one of them contained the flag.
