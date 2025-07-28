---
ai_date: 2025-04-27 05:12:43
ai_summary: Exploited client-side event handling to bypass server-side validation and send large values without triggering an error, achieving the flag by repeatedly clicking.
ai_tags:
  - xss
  - csrf
  - clickjacking
created: 2024-06-09T17:56
updated: 2025-07-14T09:46
---

>You need to get 1e20 cookies, hope you have fun clicking!

---

```js
...
socket.on('chat message', (msg) => {
	socket.emit('chat message', msg);
});

socket.on('receivedError', (msg) => {
	sessions[socket.id] = errors[socket.id]
	socket.emit('recievedScore', JSON.stringify({"value":sessions[socket.id]}));
});
...
```

The server relies on client side events for resetting the value, so we can just, *not tell the server*.

Run the following in console.

```js
socket.off('error'); // turn off error
socket.emit('click', JSON.stringify({"power":2e20, "value":send.value}));
socket.emit('click', JSON.stringify({"power":2e20, "value":send.value}));//run again
```

Easy.

```flag
bcactf{H0w_Did_Y0u_Cl1ck_S0_M4ny_T1mes_123}
```