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
```
bcactf{H0w_Did_Y0u_Cl1ck_S0_M4ny_T1mes_123}
```