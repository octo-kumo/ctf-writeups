---
created: 2024-08-04T06:13
updated: 2024-08-05T19:03
points: 135
solves: 479
---

Vigenère but more characters

```python
alphanumerical = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890!@#$%^&*(){}_?'
c = "*fa4Q(}$ryHGswGPYhOC{C{1)&_vOpHpc2r0({"
k = '5up3r_s3cr3t_k3y_f0r_1337h4x0rs_r1gh7?'
p = ''.join([alphanumerical[(alphanumerical.index(c[i]) - alphanumerical.index(k[i])) % len(alphanumerical)] for i in range(len(c))])
# my auto complete completed the entire line above lmao
print(p)
```

```flag
n00bz{4lph4num3r1c4l_1s_n0t_4_pr0bl3m}
```
