---
created: 2025-03-01T20:30
updated: 2025-03-18T02:31
solves: 22
points: 50
---

Notice the `Set-Cookie` headers.

```
set-cookie:
is_human=; Expires=Thu, 01 Jan 1970 00:00:00 GMT; Path=/
set-cookie:
is_chocolatechip=; Expires=Thu, 01 Jan 1970 00:00:00 GMT; Path=/
set-cookie:
love_cookie_so_much=; Expires=Thu, 01 Jan 1970 00:00:00 GMT; Path=/
set-cookie:
is_smart=; Expires=Thu, 01 Jan 1970 00:00:00 GMT; Path=/
```

Well apparently we can just goto `page2`.

`http://127.0.0.1:5667/page2`

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1740909568/2025/03/c2a40b762d80c0f16adcea829d9644d0.png)

Now the cookies work.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1740909585/2025/03/bd9e67a5ffe1d4244a3a600b5e5ac812.png)

```html
<form action="/human_submit" method="POST">
	<label for="pass"><h3>Enter the password:</h3></label>
	<input name="password" type="password" placeholder="password" />
	<!--  The end of the world isn't far away in this timeline... "APOCALYPSE2040" might hold the key. -->
	<input type="submit" value="Submit" />
	<p></p>
  </form>
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1740910354/2025/03/8341bd43ec46ba6919f07dd1e954f1d6.png)
Now 3 links!

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1740910445/2025/03/a89e4921bf23066df2d1996914dcbad6.png)

`1081`

```flag
ATHACKCTF{AIApocalypseIsGone}
```

eyy.
