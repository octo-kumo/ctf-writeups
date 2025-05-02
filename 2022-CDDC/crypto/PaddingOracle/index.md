---
ai_date: '2025-04-27 05:10:43'
ai_summary: Base64-encoded flag input required, likely involving a simple bruteforce
  or input validation bypass.
ai_tags:
- base64
- brute
- input-validation
created: 2024-06-11T01:17
updated: 2024-07-07T23:08
---

```text
Use any ONE of the servers below to complete the challenge. 
If you are unable to establish connection to a server of your choice, you may select another one from the list below. 
Please note that you need to connect to ONLY ONE of the servers below to solve this challenge.

nc 18.141.181.118 8573
nc 13.213.59.167 8573
nc 3.1.195.91 8573
nc 13.215.159.21 8573
nc 18.140.61.166 8573
```

From the source given, we can see that the input just have to be `{flag}` in base64

```python
try : 
    s = raw_input("Please give me base64 string:").decode('base64') 
    if not s : 
        break 
except Exception as e:
    print e 
    break 

iv = random_bytes(16)
enc = s.replace("{flag}", flag)
```

```shell
$ echo '{flag}' | base64
# e2ZsYWd9Cg==
$ nc 18.141.181.118 8573
Please give me base64 string:e2ZsYWd9Cg==
WYhjrL0sqxSFjvdpzshoA+jPAVLqQj4HHywVKCJdNFyNoYHgCyMjwCsw1mTi0Gbc
```