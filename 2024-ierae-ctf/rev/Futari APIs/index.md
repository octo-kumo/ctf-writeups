---
created: 2024-09-21T02:26
updated: 2024-09-21T02:29
---

Solved using only web tools.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1726899986/2024/09/cdfb4295e568f86103bf60909421f8e2.png)
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1726900102/2024/09/8d7c9e10cdf97c8b31e4214528b06067.png)
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1726900112/2024/09/cb554ed218e47ce9be1e41b9c212d424.png)

```flag
IERAE{Lua_1s_S0_3duc4t1onal}
```

```python [solve.py]
s=[i for i in range(29)]
s[1]=232^161
s[2]=110^43
s[3]=178^224
s[4]=172^237
s[5]=212^145
s[6]=25^98
s[7]=53^121
s[8]=63^74
s[9]=135^230
s[10]=92^3
s[11]=38^23
s[12]=250^137
s[13]=216^135
s[14]=5^86
s[15]=69^117
s[16]=226^189
s[17]=137^186
s[18]=148^240
s[19]=64^53
s[20]=130^225
s[21]=241^197
s[22]=151^227
s[23]=203^250
s[24]=179^220
s[25]=216^182
s[26]=101^4
s[27]=238^130
s[28]=61^64
print(bytes(s))
```
