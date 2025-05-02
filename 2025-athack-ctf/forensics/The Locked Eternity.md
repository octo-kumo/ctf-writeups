---
ai_date: '2025-04-27 05:25:10'
ai_summary: Found encrypted keys file requiring decryption for flag
ai_tags:
- aes
- secret
- decryption
created: 2025-03-02T00:16
tags:
- unsolved
updated: 2025-03-02T14:19
---

```bash
└─$ grep -r "QVRIQUNLQ1RGe" .

└─$ grep -r "ATHACKCTF" .
grep: ./LiveResponseData/CopiedFiles/Houssem0x1/Library/Group Containers/group.com.apple.notes/NoteStore.sqlite-wal: binary file matches
```

```flag
ATHACKCTF{Keych41nDB_&_4utoLog1n_4_the_WIN!!!}
```

Couldn't solve it, turns out theres a keys file that has to be decrypted.