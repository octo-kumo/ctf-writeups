---
created: 2025-06-06T21:09
updated: 2025-06-08T12:01
solves: 22
points: 484
---

So basically I made a three.js app that will project all the keys onto the `y=0` plane, and allow me to navigate the world, massage the quaternions etc.

By tweaking and turning the keys until they fall into somewhat of a keyboard layout, I will then substitute in the first few letters of the flag and manually guess the rest.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1749273061/2025/06/36f9d8a83e34cfd150e273ab82f301a6.png)

My guess was, well, flipped, left and right.

But since this is technically just a substitution cipher, it is a 1 to 1 mapping after all. I solved it using the almighty `dcode.fr`.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1749272904/2025/06/f652c209113f1f1713e6f048535795fe.png)

With that substituted in we can observe the keys to be flipped, but is otherwise as we would expect from a QWERTY keyboard.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1749272919/2025/06/55de737b76e10b6ffd1a8b1f68797603.png)

```flag
tjctf{i_have_friends_everywhere}
```
