---
created: 2025-03-02T08:45
updated: 2025-03-02T08:48
---

A hint?

```
:e /home/zain/hint.txt
```

```
The flag is hidden in an env variable. You have to find it by your own.
```

Hrmm...

```
:echo $ <press CTRL D>
HOME                       PATH                       SSH_CLIENT                 USER
LANG                       PWD                        SSH_CONNECTION             VIM
LOGNAME                    SHELL                      SSH_TTY                    VIMRUNTIME
OLDPWD                     SHLVL                      TERM                       YOU_HAVE_BEEN_LOOKING_FOR

:echo $YOU_HAVE_BEEN_LOOKING_FOR
```

```flag
ATHACKCTF{y0u_h4v3_35c4p3d_th3_vim_prison_0xdeadbeef}
```
