---
created: 2026-01-12T00:19
updated: 2026-01-12T00:34
title: Unrealistic Client-Side Challenge
tags:
  - unsolved
---

Doing web challenges isn't good for personal health so I was actually watching Go Go Squid instead of solving this challenge at first.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1768195211/20260112002011020.png/a639837996fe9b5fef1f29869132436b.png)

I came back to it not having much progress, so I looked into it.

I realised that if we set `motd` cookie somewhere we can do DOM injection.

So I mapped `l.yun.ng -> 127.0.0.1` and made the admin bot visit it instead.

So first report `http://127.0.0.1:5000@e.yun.ng/`, (`e` stands for evil), where a cookie `motd` is set with domain `.yun.ng` which means any subdomain should be able to read it.

Next report `http://127.0.0.1:5000@l.yun.ng:5000/motd`, the bot will visit the `/motd` page with the cookie `motd` set and our cookie is injected directly into the dom.

However the CSP is super strict `default-src 'none'; img-src http: https:; style-src 'self';` and I didn't make any progress beyond this.

Flag 1 is added to the cookies and we need to read it somehow, my idea was make `l.yun.ng -> 127.0.0.1` first, make the bot visit it, so the cookie is set on the domain `l.yun.ng` (no you can't read the cookie from a subdomain nor a parent domain), I rebind the domain to a different IP and read the cookie by making the admin bot visit it again. However DNS cache is too long for this.

The goat @aelmo solved it though (not flag 2 which is the motd stuff), with https://www.intruder.io/research/split-second-dns-rebinding-in-chrome-and-safari

hrmm.

For flag 2, we both tried many ways to steal the MOTD out of the DOM, near the end @aelmo was trying encodings but online info says its patched but after the CTF ends we realized that it was indeed encodings like what???

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1768196049/20260112003409633.png/66f1376d149125c3f5560ad32e1765be.png)

😭😭😭😭
