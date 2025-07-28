---
ai_date: 2025-04-27 05:11:36
ai_summary: Flag obtained by extracting text from elements with 'flag' class
ai_tags:
  - xss
  - queryselector
  - text-content
created: 2024-06-11T01:17
updated: 2025-07-14T09:46
---

Vising the web page we can find some interesting elements

![](https://res.cloudinary.com/kumonochisanaka/image/upload/v1718083111/2024/06/2974dab1266a7c43fdddc3fb06b12815.png)

```javascript
Array.from(document.querySelectorAll(".flag")).map(e=>e.textContent).join('')
```

And we get our flag

```flag
CDDC22{H3lL0_Spac3_tr4v3l3r5}
```