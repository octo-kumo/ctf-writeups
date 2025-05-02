---
ai_date: '2025-04-27 05:25:03'
ai_summary: 'Files contain hidden parts of flag: PART1, PART2, PART3, PART4, PART5'
ai_tags:
- payload
- data-recovery
- file-analysis
created: 2025-03-01T16:57
points: 200
solves: 25
updated: 2025-03-18T02:27
---

Just dump all files to txt.

```bash
find . -type f -print0 | while IFS= read -r -d '' file; do
  echo "Contents of $file:"
  cat "$file"
  echo
done
```

```
commit: Jester's shutdown sequence stored (PART1).
commit: Jesterâ€™s memory fragment recorded (PART2).
PART3:F1N4L_C0D3
```

Read all object files.

```bash
for obj in $(find .git/objects -type f | sed 's#.git/objects/##' | sed 's#/##'); do
    echo "Object: $obj"
    git cat-file -p $obj
    echo "-----------------"
done
```

```
PART1:K1LLC0D3_L05T
PART2:M3M0RY_H1DD3N
PART3:F1N4L_C0D3
PART4:5T45H3D_L0G
PART5:G1T0BJ3CT_M3M
```

```flag
ATHACKCTF{K1LLC0D3_L05T|M3M0RY_H1DD3N|F1N4L_C0D3|5T45H3D_L0G|G1T0BJ3CT_M3M}
```