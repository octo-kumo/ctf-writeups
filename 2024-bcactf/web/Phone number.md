---
ai_date: '2025-04-27 05:13:05'
ai_summary: Phone number input bypassed with hard-coded value, flag extracted
ai_tags:
- xss
- csrf
- rce
created: 2024-06-09T16:16
updated: 2024-06-10T23:19
---

> I was trying to sign into this website, but now it's asking me for a phone number. The way I'm supposed to input it is strange. Can you help me sign in?
>
> My phone number is 1234567890

---

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1717964216/2024/06/77652e3829385058ebc74f034c3e37b0.png)

Some digging of the code lead us to this fetch request, that seems to be the one submitting the phone number.

```js
await fetch('/flag', {
        method: "POST",
        body: InPuT.value
    }).then((res) => res.text()
```

Well, we can just run this, in the console.

```js
console.log(await fetch('/flag', {
        method: "POST",
        body: "1234567890"
    }).then((res) => res.text()))
// VM290:1 bcactf{PHoN3_num8eR_EntER3D!_17847928}
```

Simple.