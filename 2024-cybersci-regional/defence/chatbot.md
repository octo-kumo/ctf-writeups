---
ai_date: 2025-04-27 05:14:21
ai_summary: Unprotected session vulnerability and LLM access control improved
ai_tags:
  - ssrf
  - sql
  - auth-bypass
created: 2024-11-23T16:03
points: 1833
solves: 10
updated: 2025-07-14T09:46
---

> LLM is all the rage! Our intern created this chatbot poc which leverages LLMs to answer questions from voters! It was working so well that we deployed it straight to production! We are now getting reports of usual behavior on the site. We captured the traffic and created a script to re-run it at the click of the button.
>
> Please use the Attack Portal at [http://10.0.2.50](http://10.0.2.50/) to launch attacks. Use the buttons to run the traffic again and fix the issues! Note that the scripts can 1-5 minutes to fully run! The machine only has poc level resources!
>
> The chatbot website is hosted at [http://10.0.2.51](http://10.0.2.51/) You have full admin access to the machine via ssh (ssh -i {key_file> vpcadmin@10.0.2.51)
>
> `ssh -i defence_ed25519 vpcadmin@10.0.2.51`

## part 1

Session is created using only the username, which means as long as person `Dummy` has logged in, anyone can forge a valid session by base64 encoding the username `Dummy`.

I fixed this by making each session a custom json object with a random string attached.

```json
{
	"username": "Username",
	"random": <24 bytes random hex string>
}
```

## part 2

The LLM is permitted to access a lot of information, we can fix this by restricting access.

I'm not sure which part of my fixes did it, nor do I have the files now since the CTF has ended.

1. I made each user's operations be validated against their usernames in the `@tools` functions.
2. I made the SQL queries more strictly scoped, i.e. do not fetch passwords.
3. I modified some queries to satisfy the requirements of "only candidates are public" etc.

## part 3

I believe that I've actually solved this, but the attack script keeps saying "Service no longer working" and I ran out of time.