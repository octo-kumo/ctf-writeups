---
ai_date: '2025-04-27 05:11:13'
ai_summary: Morse code decryption challenge
ai_tags:
- morse
- decode
- ciphertext
created: 2024-06-11T01:17
updated: 2024-07-07T23:08
---

Opening the file we find a bunch of spaces and tabs, so...

```python
MORSE_CODE_DICT = {'A': '.-', 'B': '-...',
                   'C': '-.-.', 'D': '-..', 'E': '.',
                   'F': '..-.', 'G': '--.', 'H': '....',
                   'I': '..', 'J': '.---', 'K': '-.-',
                   'L': '.-..', 'M': '--', 'N': '-.',
                   'O': '---', 'P': '.--.', 'Q': '--.-',
                   'R': '.-.', 'S': '...', 'T': '-',
                   'U': '..-', 'V': '...-', 'W': '.--',
                   'X': '-..-', 'Y': '-.--', 'Z': '--..',
                   '1': '.----', '2': '..---', '3': '...--',
                   '4': '....-', '5': '.....', '6': '-....',
                   '7': '--...', '8': '---..', '9': '----.',
                   '0': '-----', ', ': '--..--', '.': '.-.-.-',
                   '?': '..--..', '/': '-..-.', '-': '-....-',
                   '(': '-.--.', ')': '-.--.-'}

with open('blind_for_.-.txt') as file:
    lines = file.readlines()
    decipher = ''
    for line in lines:
        line = line.replace(' ', '.').replace('\t', '-').replace('\n', '')
        decipher += list(MORSE_CODE_DICT.keys())[list(MORSE_CODE_DICT.values()).index(line)]
    print(f'CDDC22{{{decipher}}}')
```

And we get our flag

```flag
CDDC22{L1STEN-T0-THE-M0RSE-C0D3-PLE4SE}
```