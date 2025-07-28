---
ai_date: 2025-04-27 05:12:55
ai_summary: JavaScript eval with whitelist vulnerability, exploiting JSFuck to execute arbitrary code
ai_tags:
  - jsfuck
  - eval
  - arbitrary-code-execution
created: 2024-06-10T16:37
updated: 2025-07-14T09:46
---

> Hey, can you help me on this Javascript problem? Making strings is hard.

---

```js
for (let i of ["[", "]", "(", ")", "+", "!"]) {
	d = d.replaceAll(i, "");
}
...
c = eval(req.body).toString();
```

At first glance, this is a `eval` with whitelist problem.

Happens that there's a website called [JSFuck - Write any JavaScript with 6 Characters: []()!+](https://jsfuck.com/).

We will encode `Deno.readTextFileSync('flag.txt')` into `[]()+!`, and submit it to get...

The flag.

```flag
bcactf{1ava5cRIPT_mAk35_S3Nse_48129846}
```