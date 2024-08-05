---
created: 2024-06-14T16:44
updated: 2024-08-04T19:47
solves: 204
points: 380
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
