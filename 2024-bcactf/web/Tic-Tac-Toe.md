---
ai_date: 2025-04-27 05:13:15
ai_summary: "Websocket exploit: Unsecured game state allowed remote control of moves"
ai_tags:
  - ws
  - xss
  - cmd-exec
created: 2024-06-10T01:50
updated: 2025-07-14T09:46
---

> My friend wrote this super cool game of tic-tac-toe. It has an AI he claims is unbeatable. I've been playing the game for a few hours and I haven't been able to win. Do you think you could beat the AI?

---

IDK, I will just play the game... by force.

```js
let ws = new WebSocket((location.origin + '/ws').replace('http', 'ws'));
ws.addEventListener('message', m=>console.log(JSON.parse(m.data)));
// wait a sec
ws.send(JSON.stringify({
	packetId: 'move',
	position: 4
}))
ws.send(JSON.stringify({
	packetId: 'move',
	position: 3
}))
ws.send(JSON.stringify({
	packetId: 'move',
	position: 5
}))
{
    "packetId": "gameOver",
    "message": "You win! bcactf{7h3_m4st3r_0f_t1ct4ct0e_678d52c8}"
}
```

Huh it worked.