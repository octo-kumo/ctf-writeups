---
ai_date: '2025-04-27 05:11:59'
ai_summary: Writeup suggests exploiting a potential bypass of NotHandSanitizer™ to
  bypass SQL injection restrictions.
ai_tags:
- sql
- bypass
- not-hand-sanitizer
created: 2024-06-11T01:33
updated: 2024-06-11T01:34
---

> APOCALYPSE has recently implemented a security feature called NotHandSanitizer™ to secure their member login portal.
>
> We heard that there's a flag somewhere in their database, but we can't seem to find a working attack vector since SQL Injections seem impossible due to NotHandSanitizer™. Perhaps you could take a look for us?

```regexp
.*([\[\]\{\}:\\|;?!~`@#$%^&*()_+=-]|[ ]|[']|[\"]|[<]|[>]).*
```