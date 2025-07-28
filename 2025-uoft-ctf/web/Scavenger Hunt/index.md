---
ai_date: 2025-04-27 05:28:46
ai_summary: Flag found in HTML comment, headers, cookies, and source maps, requiring inspection and enumeration.
ai_tags:
  - xss
  - cookie
  - http-hdr
  - map-source
created: 2025-01-10T19:26
points: 100
solves: 499
updated: 2025-07-14T09:46
---

## part 1
Is in the HTML.

```
<!-- part 1: uoftctf{ju57_k33p_ -->
```

## part 2
Headers.

```
x-flag-part2: c4lm_4nd_
```

## part 3
Cookies.

```
flag_part3: 1n5p3c7_
```

## part 4
[34.150.251.3:3000/robots.txt](http://34.150.251.3:3000/robots.txt)

```
User-agent: *
Disallow: /hidden_admin_panel
# part4=411_7h3_
```

## part 5
[34.150.251.3:3000/styles.css](http://34.150.251.3:3000/styles.css)

```
/* p_a_r_t_f_i_v_e=4pp5_*/
```

## part 6
[Admin Panel](http://34.150.251.3:3000/hidden_admin_panel)
Accessible after changing cookie to admin.

```
Part 6: 50urc3_
```

## part 7

The previous parts say "calm and inspect all the apps source".

In `app.min.js` we find `//# sourceMappingURL=app.min.js.map`

[34.150.251.3:3000/app.min.js.map](http://34.150.251.3:3000/app.min.js.map)

```
"part7": "c0d3!!}"
```

## flag

```flag
uoftctf{ju57_k33p_c4lm_4nd_1n5p3c7_411_7h3_4pp5_50urc3_c0d3!!}
```