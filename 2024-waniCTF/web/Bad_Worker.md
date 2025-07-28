---
ai_date: 2025-04-27 05:23:57
ai_summary: Service worker detected; disabling it reveals a flag.
ai_tags:
  - service-worker
  - offline
  - flag
created: 2024-06-21T19:37
points: 120
solves: 569
updated: 2025-07-14T09:46
---

> オフラインで動くウェブアプリをつくりました。
> We created a web application that works offline.
> [https://web-bad-worker-lz56g6.wanictf.org](https://web-bad-worker-lz56g6.wanictf.org/)

On the mention of "works offline", we know its a service worker.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1719013212/2024/06/40f7cca8ff433c902a452de1c918828c.png)

And there it is, comment it out and press the button.

```flag
FLAG{pr0gr3ssiv3_w3b_4pp_1s_us3fu1}
```