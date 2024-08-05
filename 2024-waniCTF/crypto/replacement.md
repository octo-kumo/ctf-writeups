---
created: 2024-06-22T02:03
updated: 2024-08-05T09:49
points: 126
solves: 431
---

It appears that every character is replaced by their MD5 hash.

Solve by pre-computing hash table for every printable character.

```python
import hashlib
import string
import json

table = dict()
for char in string.printable:
    x = hashlib.md5(str(ord(char)).encode()).hexdigest()
    table[int(x)] = char
        
with open('my_diary_11_8_Wednesday.txt', 'r') as f:
    json_data = json.load(f)
    for item in json_data:
        print(table[item], end='')
```

```
Wednesday, 11/8, clear skies. This morning, I had breakfast at my favorite cafe. Drinking the freshly brewed coffee and savoring the warm buttery toast is the best. Changing the subject, I received an email today with something rather peculiar in it. It contained a mysterious message that said "This is a secret code, so please don't tell anyone. FLAG{13epl4cem3nt}". How strange!

Gureisya
```
