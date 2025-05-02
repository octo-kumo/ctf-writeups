---
ai_date: '2025-04-27 05:25:55'
ai_summary: Linear algebra used to calculate the password from given equations
ai_tags:
- math
- linear-algebra
- xor
created: 2025-03-01T15:01
points: 150
solves: 20
updated: 2025-03-18T02:30
---

Simple linear algebra!

```c
int __fastcall handle_client(unsigned int a1)
{
  __int64 v2[3]; // [rsp+10h] [rbp-4C0h] BYREF
  _WORD v3[5]; // [rsp+28h] [rbp-4A8h]
  __int64 v4; // [rsp+32h] [rbp-49Eh]
  char v5[64]; // [rsp+40h] [rbp-490h] BYREF
  __int64 buf[8]; // [rsp+80h] [rbp-450h] BYREF
  char v7; // [rsp+C0h] [rbp-410h] BYREF
  char v8; // [rsp+C1h] [rbp-40Fh]
  char v9; // [rsp+C2h] [rbp-40Eh]
  char v10; // [rsp+C3h] [rbp-40Dh]
  char v11; // [rsp+C4h] [rbp-40Ch]
  char v12; // [rsp+C5h] [rbp-40Bh]
  char v13; // [rsp+C6h] [rbp-40Ah]
  ssize_t v14; // [rsp+4C8h] [rbp-8h]

  strcpy((char *)buf, "Good evening Ned. Did you do what I asked you to do?\n");
  send(a1, buf, 0x36uLL, 0);
  strcpy(v5, "Hold on... before you say anything... what's the password?\n");
  send(a1, v5, 0x3CuLL, 0);
  v14 = recv(a1, &v7, 0x400uLL, 0);
  *(&v7 + v14) = 0;
  if ( v12 + v11 + v10 + v9 + v8 + v7 + v13 == 546 )
  {
    if ( v12 + v11 + v10 + v9 + v8 + v7 - v13 == 480
      && v11 + v10 + v9 + v8 + v7 - v12 + v13 == 412
      && v12 + v10 + v9 + v8 + v7 - v11 + v13 == 440
      && v12 + v11 + v9 + v8 + v7 - v10 + v13 == 312
      && v12 + v11 + v10 + v8 + v7 - v9 + v13 == 356
      && v12 + v11 + v10 + v9 + v7 - v8 + v13 == 314 )
    {
      show_flag(a1);
    }
  }
  else
  {
    v2[0] = 0x74207369202E2E2ELL;
    v2[1] = 0x2C756F7920746168LL;
    v2[2] = 0x203F7265676F5220LL;
    v3[0] = 25927;
    *(_QWORD *)&v3[1] = 0x666F2074756F2074LL;
    v4 = 0xA2E6572656820LL;
    send(a1, v2, 0x2AuLL, 0);
    sleep(1u);
  }
  return close(a1);
}
```

$$
\begin{bmatrix}
1 & 1 & 1 & 1 & 1 & 1 & 1 \\
1 & 1 & 1 & 1 & 1 & 1 & -1 \\
1 & 1 & 1 & 1 & 1 & -1 & 1 \\
1 & 1 & 1 & 1 & -1 & 1 & 1 \\
1 & 1 & 1 & -1 & 1 & 1 & 1 \\
1 & 1 & -1 & 1 & 1 & 1 & 1 \\
1 & -1 & 1 & 1 & 1 & 1 & 1 \\
\end{bmatrix}
\begin{bmatrix}
v_7 \\ v_8 \\ v_9 \\ v_{10} \\ v_{11} \\ v_{12} \\ v_{13}
\end{bmatrix}
=
\begin{bmatrix}
546 \\ 480 \\ 412 \\ 440 \\ 312 \\ 356 \\ 314
\end{bmatrix}
$$

The solution is obtained by multiplying the inverse of the coefficient matrix with the constant matrix

$$
\begin{bmatrix}
v_7 \\ v_8 \\ v_9 \\ v_{10} \\ v_{11} \\ v_{12} \\ v_{13}
\end{bmatrix}
= 
\begin{bmatrix}
1 & 1 & 1 & 1 & 1 & 1 & 1 \\
1 & 1 & 1 & 1 & 1 & 1 & -1 \\
1 & 1 & 1 & 1 & 1 & -1 & 1 \\
1 & 1 & 1 & 1 & -1 & 1 & 1 \\
1 & 1 & 1 & -1 & 1 & 1 & 1 \\
1 & 1 & -1 & 1 & 1 & 1 & 1 \\
1 & -1 & 1 & 1 & 1 & 1 & 1 \\
\end{bmatrix}^{-1}
\begin{bmatrix}
546 \\ 480 \\ 412 \\ 440 \\ 312 \\ 356 \\ 314
\end{bmatrix}
$$

This yields the ASCII values for the password characters

$$
\begin{bmatrix}
65 \\ 116 \\ 95 \\ 117 \\ 53 \\ 67 \\ 33
\end{bmatrix}
$$

```
At_u5C!
```

```python
import socket

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("127.0.0.1", 2138))
print(s.recv(1024).decode())
s.send(b"At_u5C!\n")
print(s.recv(1024).decode())
```

```flag
ATHACKCTF{lin3arSyst3mSRock!}
```