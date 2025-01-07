---
created: 2025-01-04T18:58
updated: 2025-01-06T23:22
solves: 41
points: 424
tags:
  - mem
---

All attempts with volatility failed. So I decided to just analyse the strings manually on a hex editor.

Volatility always complain about kernel mappings and plugins, maybe the memory dump was corrupted, who knows.

Naïve search of `irisctf` failed. So I decided to look for the base64 encoded flag headers.

```
irisctf
irisctf
!irisctfz
zirisctf!
!!irisctfzz
zzirisctf!!

aXJpc2N0Zg==
aXJpc2N0Zg==
IWlyaXNjdGZ6
emlyaXNjdGYh
ISFpcmlzY3Rmeno=
enppcmlzY3RmISE=
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736038695/2025/01/b504980b831a8ed2f64f71c7c2d74130.png)

```
aXJpc2N0ZntpX2FtX2FuX2
irisctf{i_am_an_
```

Nice, first guess and I've got it.

Next search for `aXJpc2N0ZntpX2FtX2FuX2`

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1736038802/2025/01/abfab0809ea5f103282f39894bfcdf9b.png)

```
aXJpc2N0ZntpX2FtX2FuX2lkaW90X3dpdGhfYmFkX21lbW9yeX0%3D
irisctf{i_am_an_idiot_with_bad_memory}7
```

```flag
irisctf{i_am_an_idiot_with_bad_memory}
```

And the challenge is solved.
