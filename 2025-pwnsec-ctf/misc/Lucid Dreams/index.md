---
created: 2025-11-16T05:15
updated: 2025-11-16T10:11
tags:
  - blood
description: yet rous’d we must be, when all such dreaming ends
points: 495
solves: 1
title: Lucid Dreams
---

We are given a file `dream.arrow`

```[dream.arrow]
00000000: 4152 524f 5731 0000 ffff ffff 5022 0000  ARROW1......P"..
00000010: 1000 0000 0000 0a00 0e00 0600 0500 0800  ................
00000020: 0a00 0000 0001 0400 1000 0000 0000 0a00  ................
00000030: 0c00 0000 0400 0800 0a00 0000 d018 0000  ................
00000040: 0400 0000 0100 0000 0c00 0000 0800 0c00  ................
00000050: 0400 0800 0800 0000 a818 0000 0400 0000  ................
00000060: 9a18 0000 7b22 696e 6465 785f 636f 6c75  ....{"index_colu
00000070: 6d6e 7322 3a20 5b7b 226b 696e 6422 3a20  mns": [{"kind":
00000080: 2272 616e 6765 222c 2022 6e61 6d65 223a  "range", "name":
00000090: 206e 756c 6c2c 2022 7374 6172 7422 3a20   null, "start":
...
```

After some google searches it turns out to be a **Apache Arrow** file, and from the readable JSON contents we can assume that it contains some sort of panda data frame.

That is indeed true.

```python
import pandas as pd
df = pd.read_feather("/content/dream.arrow")
df.columns
Index(['oh_27', 'oh_14', 'oh_32', 'c12', 'c14', 'oh_9', 'oh_13', 'oh_38',
       'oh_33', 'c9', 'oh_22', 'oh_35', 'oh_20', 'oh_18', 'c13', 'oh_11',
       'oh_17', 'c7', 'oh_15', 'oh_7', 'oh_28', 'c3', 'oh_36', 'oh_6', 'oh_2',
       'label', 'c0', 'oh_4', 'oh_21', 'oh_23', 'oh_31', 'c2', 'oh_29',
       'oh_37', 'oh_19', 'oh_30', 'oh_34', 'oh_1', 'oh_8', 'oh_0', 'c4',
       'oh_25', 'c6', 'oh_3', 'c11', 'oh_5', 'oh_24', 'c1', 'oh_26', 'c10',
       'oh_10', 'c5', 'oh_12', 'c15', 'oh_16'],
      dtype='object')
```

## naive attempt

Well this seems like some sort of dataset for an char-by-char auto complete AI, whos best at writing AI code? AI of coz.

The almighty LLMs made a simple model training code that I left in background running.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1763290327/20251116055206932.png/1d48894412c683004fc6df1420f184fa.png)

I will come back to it being solv- oh no.

The accuracy is just consistently low, and the final flag produced was...

```
flag{os_ŔIkeobuihdinr@!carlasedixlteet_eog...
```

The AI added some simple methods of cleaning up the data but whatever it is, it wasn't working coz there were still binary strings in `c14` columns 😭

Guess I need to do it myself.

## manual cleanup

> The data felt incomplete and encoded in multiple ways. If I could clean it up and learn from the patterns, perhaps the dream would reveal its meaning.

This is a very clear hint that we need to do clean up.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1763290615/20251116055655365.png/511e4c887baf25ab99717861f0c46187.png)

### characters

From just a glance, we can see that `c14` is binary, `c1` is hex, some columns seem to be normal, and all the columns have random `None` entries. Each column also appears to have consistent encoding.

```python
_df['c0'].unique()
array(['7b', '64', '67', '74', '6c', '5f', '61', '72', '76', '39', '6f',
       '6d', '65', '68', '6a', '66', '6e', '75', '63', '6b', '31', '33',
       '69', '37', '62', '7d', '73', '71', '78', '999', '77', '32', '70',
       '???', '34', '@@@', '79', '7a', '30', '38', '35', '36', None,
       'ERR'], dtype=object)
```

And after manually looking at some columns and trying to convert them I collected the bad tokens.

```python
bad_tokens = ['@@@', None, '999', '???', 'ERR']
```

We can just preprocess the whole data frame first.

```python
def p_hex(col):
  if df[col].apply(lambda x: x in bad_tokens or len(x) == 2).all():
    print(col,"is hex")
    assert df[col].apply(lambda x: x in bad_tokens or (len(x) == 2 and all(c in '0123456789abcdef' for c in x))).all()
    df[col] = df[col].apply(lambda x: ' ' if x in bad_tokens else bytes.fromhex(x).decode('utf-8'))
  elif df[col].apply(lambda x: x in bad_tokens or all(c in '01' for c in x)).all():
    print(col,"is bin")
    assert df[col].apply(lambda x: x in bad_tokens or (all(c in '01' for c in x))).all()
    df[col] = df[col].apply(lambda x: ' ' if x in bad_tokens else chr(int(x, 2)))
  elif df[col].apply(lambda x: x in bad_tokens or len(x) == 1).all():
    print(col,"is ascii")
    assert df[col].apply(lambda x: x in bad_tokens or (len(x) == 1 and all(ord(c) < 128 for c in x))).all()
    df[col] = df[col].apply(lambda x: ' ' if x in bad_tokens else x)
  elif df[col].apply(lambda x: x in bad_tokens or all(c in '0123456789' for c in x)).all():
    print(col,"is octal")
    assert df[col].apply(lambda x: x in bad_tokens or (all(c in '0123456789' for c in x))).all()
    df[col] = df[col].apply(lambda x: ' ' if x in bad_tokens else chr(int(x, 8)))
  else:
    print(col,"is unknown")
for col in df.columns:
  if col.startswith('c'):
    p_hex(col)
```

#### lower case

> I vividly remember the dream’s charset being `^[a-z0-9{}_]$`

This turns out to be really helpful because now I can clean the dataset even further, I converted all uppercase chars to lowercase and turned all chars not supposed to be there into spaces.

#### the missing c8

You might have noticed that `c8` is missing.

I thought about it and decided to just set `c8` to space, so space would mean unknown character in my dataset.

#### joined

For ease of programming lets join all the characters into a new column.

```python
cols = [f"c{i}" for i in range(16)]
df['c8'] = ' '
df["joined"] = df[cols].astype(str).agg("".join, axis=1)
```

Now that our characters have been restored, maybe we can just find the flag in there.

```python
df[df['joined'].str.contains("flag{")]['joined']
|        | joined           |
| ------ | ---------------- |
| 5822   | flag{lu4 1 dr 4m |
| 15157  | dm _{ru s_flag{  |
| 15687  | flag{lu8 d dr14m |
| 19221  | y_flag{l c1d_fre |
| 24150  | te_flag{ uc1dd7  |
| ...    | ...              |
| 492388 | _flag{l 1d_dre   |
| 498240 | flag{luc y_jre4m |
| 500203 | kjto_uki _flag{g |
| 505708 | flag{luc d_drec} |
| 506179 | eeaphtie _flag{l |
```

Well we could make up vaguely the words `flag{lucid_dream` but it seems heavily obfuscated with noise and char alterations.

I thought about doing this analytically (spoilers) but then I was like: maybe I should make an AI for this.

So I looked at `label` and `oh` to figure out what do they mean. If this is a dataset they are probably used for the training right?

### one-hot and labels

`oh` is probably `one-hot`, since the columns are just 0 or 1 mostly.

There are other values like `X` and `None` but we can just replace them with `0`, there are also rows with more than 1 active `oh_x`, I decided to just remove them, which left us with 500000 rows exactly! (pretty assuring)

So `oh_0` to `oh_38` is just a number from 0 to 38, which happens to be 39 total possible numbers, which fits 39 total chars in the charset.

```python
oh_cols = [f"oh_{i}" for i in range(39)]
for col in oh_cols:
  df[col] = df[col].apply(lambda x: '0' if x == 'X' or x == None else x)
  df[col] = df[col].astype(int)
df = df[df[oh_cols].sum(axis=1) == 1]
df["voh"] = df[oh_cols].idxmax(axis=1).str[3:].astype(int)
```

It means that `oh_x` probably maps to a character.

`label` itself is also a character, so hopefully they match up.

## randomness and madness

Turns out, no, they don't.

```python
out = []
for v in range(0, 39):
    lll=f.loc[f["voh"] == v, "label"]
    counts = lll.value_counts().head(5)
    row = {"voh": v}
    for i, (label, c) in enumerate(counts.items(), start=1):
        row[f"top{i}"] = f"{label} ({c*100/len(lll):f}%)"
    out.append(row)
table = pd.DataFrame(out).fillna(0)
table
```

|     | voh | top1       | top2      | top3     | top4     | top5     |
| --- | --- | ---------- | --------- | -------- | -------- | -------- |
| 0   | 0   | \_ (9.3%)  | e (9.0%)  | t (6.5%) | n (6.1%) | a (6.1%) |
| 1   | 1   | \_ (14.2%) | e (8.2%)  | o (7.9%) | l (7.6%) | r (5.6%) |
| 2   | 2   | \_ (12.6%) | s (9.6%)  | i (7.7%) | e (7.6%) | n (7.1%) |
| 3   | 3   | \_ (8.6%)  | e (7.4%)  | s (7.1%) | n (6.4%) | a (5.8%) |
| 4   | 4   | \_ (10.0%) | e (9.0%)  | i (6.8%) | s (6.3%) | n (6.0%) |
| 5   | 5   | \_ (14.0%) | e (10.4%) | a (8.0%) | i (6.7%) | s (5.6%) |
| 6   | 6   | \_ (14.4%) | s (9.8%)  | e (8.2%) | o (5.2%) | r (4.9%) |
| 7   | 7   | \_ (9.3%)  | e (8.4%)  | a (7.7%) | t (6.6%) | i (6.1%) |

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1763292895/20251116063455136.png/0fd4f6295e6907ddd63dfe582bbca55d.png)

Instead of having some sort of 1-to-1 mapping between `label` and `oh`, they seem to be completely uncorrelated.

There is a bit of exception to this, if I apply a filter that allows only rows with `flag{` in them, some of the mapping are quite obvious (high correlation).

```
"  m         e         4      s         r c"
"0123456789 0123456789 0123456789 012345678"
```

But that's it.

I got stuck here asking the author about the mapping issue but got no answers on this, coz other teams are also close.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1763292662/20251116063102302.png/940ba60bcca5c7a25748a4743e470834.png)

## the other way

Ok let's ignore all that `oh` and `label` madness, they are effectively random.

We can just focus on the characters, remember how after we reconstructed the characters we could see pieces of the flag, sort of?

Let's assume that the flags are in pieces, altered slightly, and scattered in the dataset.

What if we do a sort of next-character-lookup, where we analyse the entire dataset for the most probable next character after `flag{`? 🤔

### substring fuzzy hamming code

For simplicity lets start with `flag{`, and we need to find the next character.

First we need to actually know if `flag{` exist in each row, but since there might be alterations, we can use the minimal hamming distance over a sliding window, then compute a score. This would hopefully save it from the random char alterations.

For example if `flag{` was changed to `fxag{`, this should still work, however it won't work for `fag{`, which shouldn't happen / should be quite rare in the dataset as I don't see it.

A score of 0 means the substring is definitely not found, while score of 1 means the exact substring has been found.

We also need to know the next character so it would be returned as well.

The score would decide how probably-correct that next character is.

```python
def fuzzy_match(string1, string2):
  n = len(string1)
  m = len(string2)
  if m > n:
	return 0.0, None
  min_diff = float('inf')
  best_i = -1
  for i in range(n - m + 1):
	diff = sum(1 for j in range(m) if string1[i + j] != string2[j])
	if diff < min_diff:
	  min_diff = diff
      best_i = i
  score = (1.0 - (min_diff / m))**4
  next_char = string1[best_i + m] if best_i + m < n else None
  return score, next_char
```

### autocomplete the flag

As the flag itself grows longer and longer, we might want to only consider the last $k$ characters because otherwise we would find a lot less rows in the dataset which could increase uncertainty of the final prediction.

We will take score-weighted average to find the top 3 candidates for the next character, and just repeat after append.

```python
flag="flag{"
while True:
  df_c = df
  scores, next_chars = zip(*df_c.apply(lambda x: fuzzy_match(x['joined'], flag[-12:]), axis=1))
  df_c['score'] = scores
  df_c['next_char'] = next_chars
  thres=0.2
  filtered_df = df_c[df_c['score'] > thres].copy()
  total_scores = filtered_df.groupby('next_char')['score'].sum()
  top_3 = total_scores.sort_values(ascending=False).head(3)
  print(top_3)
  flag += top_3.index[0]
  print(flag)
  if top_3.index[0] == '}':
    break
```

And it works!

Although slightly slow, it is churning out the flag, char-by-char.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1763293724/20251116064844780.png/5a9ae55bc300efedb25f53c257355c5b.png)

But `oh` and `label` were not used. Hrmm.

Welp the author said I'll know after the event.

```
next_char
l    114.7168
      41.2032
e      4.6864
Name: score, dtype: float64
flag{l
next_char
u    72.189815
     26.432099
t     3.858025
Name: score, dtype: float64
flag{lu
next_char
c    78.429821
     27.627239
v     2.139942
Name: score, dtype: float64
...
flag{luc1d_dre4ms_wher3_consci0usness_bl3nds_with_unc0nsc1ous_et3rn4l
next_char
l    111.2
       8.7
6      3.8
Name: score, dtype: float64
flag{luc1d_dre4ms_wher3_consci0usness_bl3nds_with_unc0nsc1ous_et3rn4ll
next_char
y    123.7
      18.0
m      3.3
Name: score, dtype: float64
flag{luc1d_dre4ms_wher3_consci0usness_bl3nds_with_unc0nsc1ous_et3rn4lly
next_char
}    128.4
      10.4
{      5.3
Name: score, dtype: float64
flag{luc1d_dre4ms_wher3_consci0usness_bl3nds_with_unc0nsc1ous_et3rn4lly}
```

And there it is! The flag!

### the flag (?)

Finally, submit and blo-

```flag
flag{luc1d_dre4ms_wher3_consci0usness_bl3nds_with_unc0nsc1ous_et3rn4lly}
```

It is ...incorrect! Looks so right though.

Hold on, it is just 1 char off!

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1763293956/20251116065236076.png/7e68e21095b4d3f5a6e879a2598e4aed.png)

After running the script again a few times I got it!

```flag
flag{luc1d_dre4ms_wher3_consci0usne5s_bl3nds_with_unc0nsc1ous_et3rn4lly}
```

Got the blood too.

I'm probably going to be soon unconscious now that its 6AM.
