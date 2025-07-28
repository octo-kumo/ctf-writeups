---
ai_date: 2025-04-27 05:15:22
ai_summary: "Decrypted text reveals flag: DUCTF{when_in_doubt_xort_it_out}"
ai_tags:
  - xor
  - frequency-analysis
  - english
created: 2024-07-05T23:26
points: 247
solves: 44
tags:
  - fav
  - freq-analysis
title: Three Line Crypto
updated: 2025-07-14T09:46
---

A cipher with only three lines?
## Analysis

After playing around with the cipher I realized that

- Any word after a space would have the same cipher-text (`SPACE % 16 == 0`), no matter where they are.
- No matter what came before the given word, when the encryption encounters a given word, the next stream of keys would be fixed, resulting in identical cipher text.
- I only have to determine 16 unknown bytes.
- The passage is in english.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1720250627/2024/07/90220e825c3508d1fc0f611d0485a8f0.png)
<small>small attempt at key guessing</small>
## Counting the bytes

One idea quickly came to mind, to count the frequency of binary bytes and pair them to words.

```python
def find_most_common_sequences(raw, sequence_length=3):
	counter = Counter([raw[i:i+sequence_length] for i in range(len(raw) - sequence_length + 1)])
	most_common = counter.most_common(top_n)
```

For each sequence, we will let it propose a set of key values for us. The set of key values may not cover the entire 16 slots, but it's ok. Only the proposed values are needed for this word to be successfully encrypted into the sequence chosen.

```
plain  t  h  e ' '
key i  0  4  8  5
ciphr  A  B  C  D
```

For example, we can start with `the`, the most common word, and a possible sequence `ABCD` (binary). At each position we propose a key value for the slot specified by the last plain text character.
1. `k[0] <= 't' ^ A`
2. `k[4] <= 'h' ^ B`
3. `k[8] <= 'e' ^ C`
4. `k[5] <= ' ' ^ D`

However, if in the middle of a sequence, a different key is proposed for the same key slot, and there is a conflict, the sequence is invalid and discarded.

```python [counter.py]
def do_for_word(encrypted, word, top_n=None):
    ...

with open("passage.enc.txt", "rb") as r:
    raw = r.read()
do_for_word(raw, b'the ')
do_for_word(raw, b'there')
do_for_word(raw, b'DUCTF{')
```

The left most column is the number of occurrences found for that specific sequence, higher the better. The 16 columns that follows are the proposed keys.

```
         0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
         word b'the '
   66  -103-------------168--78---------110----------------------------
   28  -114-------------151-168---------106----------------------------
   16  -252-------------123--43---------165----------------------------
   16  --25-------------151-168---------106----------------------------
         word b'there'
   16  -197-----237-----110-125---------154----------------------------
   14  -252------11-----123-121---------165----------------------------
   10  -103-----113-----168--28---------110----------------------------
   10  -118-----165------71-238---------134----------------------------
         word b'DUCTF{'
    7  --70---------200-122-160----------------------------------------
    6  -128---------148-248--80----------------------------------------
    6  --69---------118-177-131----------------------------------------
    5  --75---------148-221--80----------------------------------------
pos 0 Counter({103: 2, 252: 2, 114: 1, 25: 1, 197: 1, 118: 1, 70: 1, 128: 1, 69: 1, 75: 1})
```

At the end I print out the detailed statistics for a specific key slot.

Since the jump between `#1` and `#2` for `'the '` is huge, we can safely assume those keys are correct. i.e. `key[0]=103` etc.

I will update the code to set those keys on initialization, so sequences that do not follow the initial key sequence will be automatically rejected.

```python
def init_key(keys=None):
    if keys == None:
        keys = [-1]*16
    keys[0] = 103
    return keys
```

## Cracking the key

With the function to count and guess keys in place, I will feed it the most common words.

```python [counter.py]
with open("words.txt", "r") as r:
    words_all = r.read()
    words = words_all.splitlines()[:1000]

for word in words:
    do_for_word(raw, word.encode())
```

Parts of the proposed keys that are identical to the default key are hidden for convenience.

```
...
   66  ----------------------------------------------------------------
    1  -------------------------------------175------------------------
         word b'been'
   66  ----------------------------------------------------------------
         word b'now'
   82  ----------------------------------------------------------------
    4  ---------------------------------------------------------107-161
    2  ----------------------------------------------------------97--78
    2  ---------------------------------------------------------101-141
         word b'find'
   66  ----------------------------------------------------------------
    7  -------------------------------------174------------------70----
    2  -------------------------------------185-----------------255----
    2  -------------------------------------185-----------------233----
         word b'any'
   82  ----------------------------------------------------------------
   28  -----145-------------------------------------------------118----
    4  -----140--------------------------------------------------99----
    3  -----141-------------------------------------------------242----
         word b'new'
   82  ----------------------------------------------------------------
    4  ----------------------------------------------------------97----
    2  ---------------------------------------------------------107----
    2  ---------------------------------------------------------111----
pos 1 Counter({145: 12, 140: 5, 141: 5, 139: 4, 144: 3, 114: 2, 120: 2, 150: 2, 137: 2, 134: 2, 147: 2, 143: 2, 128: 2, 131: 2, 162: 1, 151: 1, 149: 1, 146: 1, 135: 1, 155: 1, 142: 1, 158: 1, 168: 1, 157: 1, 129: 1, 148: 1, 190: 1, 187: 1, 159: 1, 165: 1, 123: 1, 117: 1, 169: 1})
```

For example, `145` seems to be the best key for position 1, I will update the fixed key in code and run it for the next position.
## Verifying the key

*After about half an hour of perfecting my script and trying keys.*

Let's say we have a full set of proposed keys, how good is it?
We can grade it by looking for unprintable characters.

```python [verifier.py]
l = []
for i, c in enumerate(p):
    if c not in string.printable.encode() and chr(p[i-1]) in string.printable and chr(p[i-2]) in string.printable:
        l.append(p[i-1] % 16)

print(Counter(l).most_common(5))
# [(13, 57), (8, 35), (9, 34), (14, 31), (5, 25)]

with open("passage.dec.txt", "wb") as w:
    w.write(p)
```

Now we check the file for incorrect decryption.

```
What makes the cornfield smile; beneath what star
W†yˆ1±›-ÿ¬@ï¬Gîaçß˜æž¬)xRn the sod
Už£0­‰4-³Æ¾{R9e vine; how tend the steer;
M­öY£Â#"òéG$`cattle-keeping, or what proof
UŠ›Øh‹«ï½›ÜKN¯_”6es for thrifty bees;
```

We can see that most of the text has been decrypted, but every character after a newline is wrong.
Well `'\n' % 16 == 10`, so we can determine that the 10th key is incorrect, we can mark the current value as wrong and run `counter.py` again.

## Securing the key
For my own convenience, I have also made a small script that I can paste in chunks of plaintext I believe to be 100% correct, it will give me the key indexes that are correct and no longer have to be tried.

```python [safe_index.py]
st = "..."
i = []
for c in st:
    i.append(ord(c) % 16)
i = list(set(i))
i.sort()
print(i)
```

## What is the flag

I determined the key to be `[103, 145, 232, 58, 168, 78, 141, 244, 110, 165, 44, 207, 145, 19, 107, 162]` or `b'g\x91\xe8:\xa8N\x8d\xf4n\xa5,\xcf\x91\x13k\xa2'`.

```txt [passage.dec.txt]
What makes the cornfield smile; beneath what star
Maecenas, it is meet to turn the sod
Or marry elm with vine; how tend the steer;
What pains for cattle-keeping, or what proof
Of patient trial serves for thrifty bees;
Such are my themes.

O universal lights
Most glorious! ye that lead the gliding year
Along the sky, Liber and Ceres mild,
If by your bounty holpen earth once changed
Chaonian acorn for the plump wheat-ear,
And mingled with the grape, your new-found gift,
The draughts of Achelous; and ye Fauns
To rustics ever kind, come foot it, Fauns
And Dryad-maids together; your gifts I sing.
And thou, for whose delight the war-horse first
Sprang from earth's womb at thy great trident's stroke,
Neptune; and haunter of the groves, for whom
Three hundred snow-white heifers browse the brakes,
The fertile brakes of Ceos; and clothed in power,
Thy native forest and Lycean lawns,
Pan, shepherd-god, forsaking, as the love
Of thine own Maenalus constrains thee, hear
And help, O lord of Tegea! And thou, too,
Minerva, from whose hand the olive sprung;
And boy-discoverer of the curved plough;
And, bearing a young cypress root-uptorn,
Silvanus, and Gods all and Goddesses,
Who make the fields your care, both ye who nurse
The tender unsown increase, and from heaven
Shed on man's sowing the riches of your rain:
Of which one is DUCTF{when_in_doubt_xort_it_out};
And thou, even thou, of whom we know not yet
What mansion of the skies shall hold thee soon,
Whether to watch o'er cities be thy will,
Great Caesar, and to take the earth in charge,
That so the mighty world may welcome thee
Lord of her increase, master of her times,
Binding thy mother's myrtle round thy brow,
Or as the boundless ocean's God thou come,
Sole dread of seamen, till far Thule bow
Before thee, and Tethys win thee to her son
With all her waves for dower; or as a star
Lend thy fresh beams our lagging months to cheer,
Where 'twixt the Maid and those pursuing Claws
A space is opening; see! red Scorpio's self
His arms draws in, yea, and hath left thee more
Than thy full meed of heaven: be what thou wilt-
For neither Tartarus hopes to call thee king,
Nor may so dire a lust of sovereignty
E'er light upon thee, howso Greece admire
Elysium's fields, and Proserpine not heed
Her mother's voice entreating to return-
Vouchsafe a prosperous voyage, and smile on this
My bold endeavour, and pitying, even as I,
These poor way-wildered swains, at once begin,
Grow timely used unto the voice of prayer.
In early spring-tide, when the icy drip
Melts from the mountains hoar, and Zephyr's breath
Unbinds the crumbling clod, even then 'tis time;
Press deep your plough behind the groaning ox,
And teach the furrow-burnished share to shine.
That land the craving farmer's prayer fulfils,
Which twice the sunshine, twice the frost has felt;
Ay, that's the land whose boundless harvest-crops
Burst, see! the barns.

But ere our metal cleave
An unknown surface, heed we to forelearn
The winds and varying temper of the sky,
The lineal tilth and habits of the spot,
What every region yields, and what denies.
Here blithelier springs the corn, and here the grape,
There earth is green with tender growth of trees
And grass unbidden. See how from Tmolus comes
The saffron's fragrance, ivory from Ind,
From Saba's weakling sons their frankincense,
Iron from the naked Chalybs, castor rank
From Pontus, from Epirus the prize-palms
O' the mares of Elis.
Such the eternal bond
And such the laws by Nature's hand imposed
On clime and clime, e'er since the primal dawn
When old Deucalion on the unpeopled earth
Cast stones, whence men, a flinty race, were reared.
Up then! if fat the soil, let sturdy bulls
Upturn it from the year's first opening months,
And let the clods lie bare till baked to dust
By the ripe suns of summer; but if the earth
Less fruitful just ere Arcturus rise
With shallower trench uptilt it- 'twill suffice;
There, lest weeds choke the crop's luxuriance, here,
Lest the scant moisture fail the barren sand.
Then thou shalt suffer in alternate years
The new-reaped fields to rest, and on the plain
A crust of sloth to harden; or, when stars
Are changed in heaven, there sow the golden grain
Where erst, luxuriant with its quivering pod,
Pulse, or the slender vetch-crop, thou hast cleared,
And lupin sour, whose brittle stalks arise,
A hurtling forest. For the plain is parched
By flax-crop, parched by oats, by poppies parched
In Lethe-slumber drenched. Nathless by change
The travailing earth is lightened, but stint not
With refuse rich to soak the thirsty soil,
And shower foul ashes o'er the exhausted fields.
Thus by rotation like repose is gained,
Nor earth meanwhile uneared and thankless left.
Oft, too, 'twill boot to fire the naked fields,
And the light stubble burn with crackling flames;
Whether that earth therefrom some hidden strength
And fattening food derives, or that the fire
Bakes every blemish out, and sweats away
Each useless humour, or that the heat unlocks
New passages and secret pores, whereby
Their life-juice to the tender blades may win;
Or that it hardens more and helps to bind
The gaping veins, lest penetrating showers,
Or fierce sun's ravening might, or searching blast
Of the keen north should sear them. Well, I wot,
He serves the fields who with his harrow breaks
The sluggish clods, and hurdles osier-twined
Hales o'er them; from the far Olympian height
Him golden Ceres not in vain regards;
And he, who having ploughed the fallow plain
And heaved its furrowy ridges, turns once more
Cross-wise his shattering share, with stroke on stroke
The earth assails, and makes the field his thrall.
```

And we have our flag `DUCTF{when_in_doubt_xort_it_out}`.
## Solve Script

```python [verifier.py]
import string
from typing import Counter


with open("passage.enc.txt", "rb") as r:
    raw = r.read()


def dec(i, plain, cipher, key):
    ki = (plain[i-1] if i > 0 else 0) % 16
    return bytes([key[ki] ^ cipher[i]])


def init_key(keys):
    keys[0] = 103  # CORRECT
    keys[1] = 145  # CORRECT
    keys[2] = 232  # CORRECT
    keys[3] = 58  # CORRECT
    keys[4] = 168  # CORRECT
    keys[5] = 78  # CORRECT
    keys[6] = 141  # CORRECT
    keys[7] = 244  # CORRECT
    keys[8] = 110  # CORRECT
    keys[9] = 165  # CORRECT
    keys[10] = 44  # CORRECT
    keys[11] = 207  # CORRECT
    keys[12] = 145  # CORRECT
    keys[13] = 19  # CORRECT
    keys[14] = 107  # CORRECT
    keys[15] = 162  # CORRECT
    return keys


keys = init_key([0]*16)

p = b''
for i in range(len(raw)):
    p += dec(i, p, raw, keys)

l = []
for i, c in enumerate(p):
    if c not in string.printable.encode() and chr(p[i-1]) in string.printable and chr(p[i-2]) in string.printable:
        l.append(p[i-1] % 16)

print(Counter(l).most_common(5))

with open("passage.dec.txt", "wb") as w:
    w.write(p)
```

```python [counter.py]
from typing import Counter


def key_to_str(key):
    return ''.join([f'{i:4}' if i != -1 else ' '*4 for i in key])


def init_key(keys=None):
    if keys == None:
        keys = [-1]*16
    keys[0] = 103  # CORRECT
    keys[1] = 145  # CORRECT
    keys[2] = 232  # CORRECT
    keys[3] = 58  # CORRECT
    keys[4] = 168  # CORRECT
    keys[5] = 78  # CORRECT
    keys[6] = 141  # CORRECT
    keys[7] = 244  # CORRECT
    keys[8] = 110  # CORRECT
    keys[9] = 165  # CORRECT
    # keys[10] = 44  # 145
    # keys[11] = 207  # CORRECT
    # keys[12] = 145  # CORRECT
    # keys[13] = 19  # CORRECT
    # keys[14] = 107  # CORRECT
    # keys[15] = 162  # CORRECT
    return keys


pos_i = 10  # position of the key to focus on
pos_mis = []  # counter

print(' '*5, ''.join([f'{i:4}' if i != -1 else ' '*4 for i in range(16)]))


def do_for_word(encrypted, word, top_n=None):
    print('\t', "word", word)
    curr_key = key_to_str(init_key())

    def find_most_common_sequences(raw, sequence_length=3):
        counter = Counter([raw[i:i+sequence_length] for i in range(len(raw) - sequence_length + 1)])
        most_common = counter.most_common(top_n)
        lim = 4  # max number of sequences to print
        keys = []  # to avoid duplicates
        for seq, count in most_common:
            key = init_key()
            last_plain = 0  # starting from space
            for i, c in enumerate(word):
                ki = last_plain % 16  # key index
                propose = c ^ seq[i]  # propose key for key index
                # if key is already set and it's different reject
                # (since we are counting most common sequences, it's likely to be correct)
                if key[ki] != -1 and key[ki] != propose:
                    break
                key[ki] = propose  # set key
                last_plain = c
            _key = key_to_str(key)  # key as string
            if _key not in keys and (_key != curr_key or lim == 4):  # avoid duplicates
                print(f"{count:5} ", ''.join([c if d != c else '-' for d, c in zip(curr_key, _key)]))
                if key[pos_i] != -1:  # counter for focus key
                    pos_mis.append(key[pos_i])
                lim -= 1
                if lim == 0:
                    break
                keys.append(_key)
    find_most_common_sequences(encrypted, len(word))


with open("passage.enc.txt", "rb") as r:
    raw = r.read()
do_for_word(raw, b'the ')
do_for_word(raw, b'there')
do_for_word(raw, b'DUCTF{')

with open("words.txt", "r") as r:
    words_all = r.read()
    words = words_all.splitlines()[:1000]

for word in words:
    do_for_word(raw, word.encode())

print("pos", pos_i, Counter(pos_mis))
```

## Comments

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1720250682/2024/07/2d4c46a5522d7bbe96e922503cf0e552.png)

It was one hell of a ride, from thinking its just a simple XOR cycling key cipher to frequency analysis to almost giving up to finally seeing light (image above).

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1720250891/2024/07/f76101b4a7a2042fdec17699967cc56d.png)

My favourite challenge this CTF definitely.

Very fun, would do again.