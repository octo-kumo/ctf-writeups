---
ai_date: '2025-04-27 05:24:54'
ai_summary: Found a flag in the MFT entry with creation time and content hint.
ai_tags:
- mft
- timestamp
- data
created: 2025-03-01T16:51
points: 250
solves: 19
updated: 2025-03-18T02:27
---

Open file with MFT Explorer.

![image.png](https://res.cloudinary.com/kumonochisanaka/image/upload/v1740865883/2025/03/321cc1015588c8c01fc21a3e0d37d625.png)

The secret file is at `\Users\relizane\Desktop\important\secret.txt.txt`.

```
[00000244-0000000D, Entry-seq #: 0x244-0xD, Offset: 0x91000, Flags: InUse, Log Sequence #: 0x2140BCC0, Mft Record To Base Record: Entry/seq: 0x0-0x0
Reference Count: 0x2, Fixup Data: Expected: 05-00 Fixup Actual: 33-72|00-00 (Fixup OK: True)

**** STANDARD INFO ****
Type: StandardInformation, Attribute #: 0x0, Size: 0x60, Content size: 0x48, Name size: 0x0, Content offset: 0x18, Resident: True

Flags: Archive, Max Version: 0x0, Flags 2: None, Class Id: 0x0, Owner Id: 0x0, Security Id: 0x73A, Quota Charged: 0x0 
Update Sequence #: 0xF93F178

Created On:		2025-01-25 06:41:21.8728907
Content Modified On:	2025-01-25 06:50:57.7326737
Record Modified On:	2025-01-25 06:50:57.7326737
Last Accessed On:	2025-01-25 06:50:57.7326737

**** FILE NAME ****
Type: FileName, Attribute #: 0x5, Size: 0x78, Content size: 0x5A, Name size: 0x0, Content offset: 0x18, Resident: True

File name: SECRET~1.TXT (Length: 0xC)
Flags: Archive, Name Type: Dos, Reparse Value: 0x0, Physical Size: 0x0, Logical Size: 0x0
Parent Mft Record: Entry/seq: 0x16685-0x2

Created On:		2025-01-25 06:41:21.8728907
Content Modified On:	2025-01-25 06:41:21.8728907
Record Modified On:	2025-01-25 06:41:21.8728907
Last Accessed On:	2025-01-25 06:41:21.8728907


**** FILE NAME ****
Type: FileName, Attribute #: 0x4, Size: 0x78, Content size: 0x5E, Name size: 0x0, Content offset: 0x18, Resident: True

File name: secret.txt.txt (Length: 0xE)
Flags: Archive, Name Type: Windows, Reparse Value: 0x0, Physical Size: 0x0, Logical Size: 0x0
Parent Mft Record: Entry/seq: 0x16685-0x2

Created On:		2025-01-25 06:41:21.8728907
Content Modified On:	2025-01-25 06:41:21.8728907
Record Modified On:	2025-01-25 06:41:21.8728907
Last Accessed On:	2025-01-25 06:41:21.8728907


**** OBJECT ID ****
Type: VolumeVersionObjectId, Attribute #: 0x6, Size: 0x28, Content size: 0x10, Name size: 0x0, Content offset: 0x18, Resident: True

Object Id: 8552a51d-8e03-11ef-87b5-000c29ae9287
	Object Id MAC: 00:0c:29:ae:92:87:
	Object Id Created On: 2024-10-19 10:18:44.4606749
Birth Volume Id: 00000000-0000-0000-0000-000000000000
Birth Object Id: 00000000-0000-0000-0000-000000000000
Domain Id: 00000000-0000-0000-0000-000000000000

**** DATA ****
Type: Data, Attribute #: 0x1, Size: 0x70, Content size: 0x57, Name size: 0x0, Content offset: 0x18, Resident: True

Resident Data
Data: 2D-2D-3E-20-2D-2D-3E-20-57-65-6C-63-6F-6D-65-20-74-6F-20-41-54-48-41-43-4B-43-54-46-20-32-6B-32-35-3B-20-41-54-48-41-43-4B-43-54-46-7B-4E-54-46-24-5F-4D-34-73-37-33-72-5F-66-69-4C-33-5F-54-34-62-4C-33-21-21-21-7D-0D-0A-2D-2D-3E-20-48-6F-75-73-73-65-6D-30-78-31

ASCII: --> --> Welcome to ATHACKCTF 2k25; ATHACKCTF{NTF$_M4s73r_fiL3_T4bL3!!!}
--> Houssem0x1
Unicode: ⴭ‾ⴭ‾敗捬浯⁥潴䄠䡔䍁䍋䙔㈠㉫㬵䄠䡔䍁䍋䙔乻䙔弤㑍㝳爳晟䱩弳㑔䱢ℳ℡ൽⴊ㸭䠠畯獳浥砰�

]
```

## part 1

> Done by GPT

- **MFT Entry Number:** The header shows “Entry-seq #: 0x244-0xD”. The entry part “0x244” in hexadecimal converts to **580** in decimal.
- **File Size:** The DATA attribute shows “Content size: 0x57”. In decimal, 0x57 equals **87** bytes.
- **Creation Time:** The creation timestamp (from both the STANDARD INFO and FILE NAME attributes) is “2025-01-25 06:41:21.8728907”. Dropping the fractional seconds gives **2025-01-25 06:41:21**.

```flag
ATHACKCTF{580_87_2025-01-25 06:41:21}
```

## part 2

MFT Explorer supports just reading the contents.

```flag
ATHACKCTF{NTF$_M4s73r_fiL3_T4bL3!!!}
```