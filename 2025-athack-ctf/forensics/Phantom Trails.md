---
created: 2025-03-01T17:19
updated: 2025-03-01T17:56
---

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1740867597/2025/03/d403118c5aface90d6a1de69ac2d76c0.png)

Files of the zip, sorted by modification date.

## ApplicationConfig.json

```json
{
	"IsActive": true,
	"Name": "execute",
	"Path": "%Windows%\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
	"Args": "-c \"$p=[System.Text.Encoding]::UTF8.GetString([Convert]::FromBase64String('IlYxNF9SM2cxJFJ5XzRuRF9MMEwhfSIgLWpvaW4gIiI'));Invoke-Expression $p\"",
	"OutputExtension": "",
	"Extensions": "",
	"HiddenWindow": true,
	"DeleteInputFile": false
  }
```

The very first file is suspicious.

```
"V14_R3g1$Ry_4nD_L0L!}" -join ""
```

## where_am_hiding.txt

```
ZmxhZ3tmNGtlX2ZsNGd9 -> flag{f4ke_fl4g}
```

## first part.

Couldn't find it, so I cheated.

```bash
$ grep -r "QVRIQUNLQ1RGe" .
grep: ./Windows/System32/config/SOFTWARE: binary file matches
```

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1740869753/2025/03/6ade6940d987839111df5f8d903061ce.png)

```flag
ATHACKCTF{P3rS1$TEnC3_gH0$T_V14_R3g1$Ry_4nD_L0L!}
```
