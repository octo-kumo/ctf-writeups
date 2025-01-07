---
created: 2025-01-04T17:47
updated: 2025-01-06T23:22
solves: 31
points: 457
tags:
  - json-log
  - log
---

> Here at EI Corp, ethics are our top priority! That's why our IT team was shocked when we got a knock from our ISP informing us that someone on our computer network was involved in some illegitimate activity. Who would do that? Don't they know that's illegal?
>
> Our ISP's knocking (and so is HR), and we need someone to hold accountable. Can you find out who committed this violation?
>
> Flag format: `irisctf{username}` (e.g. `irisctf{jdoe}`). Usernames are case-sensitive.

Much is the same as the first challenge.

```
10.17.160.93 has a unique visit to ubuntu.com
10.34.235.211 has a unique visit to bp0.blogger.com
10.18.16.253 has a unique visit to lexpress.fr
10.66.110.104 has a unique visit to bmj.com
1021321597 has a unique visit to deloitte.com
10.18.21.121 has a unique visit to thepiratebay.org
10.18.21.121 has a unique visit to generic-illicit-activities-hub.org
IP 10.18.21.121 is assigned to MAC de:ad:be:ef:ca:fe
```

It's quite obvious we are looking for `10.18.21.121`, but the filtered logs don't work (they don't contain any identifying info).

`10.18.21.121` has mac address `de:ad:be:ef:ca:fe` which is obviously fabricated.

How are we supposed to find the person?

## IP to name look up

From the first challenge we know all the names are revealed in those fancy SSO packets.

```python
if '_raw' in data and 'SSO' in data['_raw']:
	_raw = data['_raw'].split("|")
	ip = _raw[6]
	name = _raw[9]
	if ip not in ip_to_name:
		ip_to_name[ip] = []
	if name not in ip_to_name[ip]:
		ip_to_name[ip].append(name)
```

This is just added for ease of solve sake.

## IP holding time

I had the idea to check concurrently connected users, by identifying the time frames that they are connected and are assigned IP addresses.

This is assuming the attacker would `DHCPRELEASE` his own IP first, then `DHCPREQUEST` the sus IP.

However, for whatever reason, about $13\%$ of all users don't share collisions with `10.18.21.121`.

That's too many users to check.

```python
if 'opcode' in data and data['opcode'] == 'DHCPREQUEST':
	mac = data['chaddr']
	addr = data['riaddr']
	if mac not in ip_hold_time:
		ip_hold_time[mac] = []
	if ip_hold_time[mac] and ip_hold_time[mac][-1]['end'] is None:
		pass
	else:
		ip_hold_time[mac].append({
			'start': log['_time'],
			'end': None,
			'addr': addr
		})
if 'opcode' in data and data['opcode'] == 'DHCPRELEASE':
	mac = data['chaddr']
	if mac not in ip_hold_time:
		print(f"User {mac} has no start time")
	else:
		ip_hold_time[mac][-1]['end'] = log['_time']
```

## IP connections

Perhaps the target communicates in someway with the sus IP?

```python
if 'src_ip' in data and 'dest_ip' in data:
	src = data['src_ip']
	dest = data['dest_ip']
	if src not in connections:
		connections[src] = []
	if dest not in connections[src]:
		connections[src].append(dest)
	if dest not in connections:
		connections[dest] = []
	if src not in connections[dest]:
		connections[dest].append(src)
```

Turns out **all** clients only communicate with `10.18.0.2` and `255.255.255.255`.

```json
"10.18.21.121": [
	"10.18.0.2",
	"255.255.255.255"
]
```

## Multi-IP mac addresses

As I was looking around I realised that only one mac address had two IPs.

```python
for mac, times in ip_hold_time.items():
    all_addr = list(set([time['addr'] for time in times]))
    if len(all_addr) > 1:
        print(f"mac {mac} has multiple addresses: {all_addr}")
        for addr in all_addr:
            print(f"{addr} -> {ip_to_name.get(addr, 'Unknown')}")
```

```
mac 53:75:56:a7:98:8f has multiple addresses: ['10.18.13.187', '10.17.161.10']
10.18.13.187 -> Unknown
10.17.161.10 -> ['mhammond']
```

And it was the answer.
### flag

```flag
irisctf{mhammond}
```

I also found the IP randomly by the second last IP in the IP list.

```json
{
	"...": "...",
    "10.18.13.187": "53:75:56:a7:98:8f",
    "10.18.21.121": "de:ad:be:ef:ca:fe"
}
```

The sus IP was the last to be assigned, and the second last assigned IP is used by the attacker.

Not sure about the intended solution.
### script

```python
import json
import re
from datetime import datetime
mac_to_user = dict()
user_visited = dict()
ip_to_mac = dict()
ip_to_name = dict()
ip_hold_time = dict()
connections = dict()

mac = '10.18.21.121'
with open('logs.json') as f:
    for line in f:
        log = json.loads(line)
        data = log.get('data', {})
        if '_raw' in data and 'SSO' in data['_raw']:
            _raw = data['_raw'].split("|")
            ip = _raw[6]
            name = _raw[9]
            if ip not in ip_to_name:
                ip_to_name[ip] = []
            if name not in ip_to_name[ip]:
                ip_to_name[ip].append(name)
        # store the association between MAC and username
        if 'username' in data and 'usermac' in data:
            if data['usermac'] in mac_to_user and mac_to_user[data['usermac']] != data['username']:
                print(f"MAC {data['uermac']} has multiple users: {mac_to_user[data['usermac']]} and {data['username']}")
            mac_to_user[data['usermac']] = data['username']

        # DHCPACK means the IP is assigned to the MAC
        if 'opcode' in data and data['opcode'] == 'DHCPACK':
            if data['ciaddr'] in ip_to_mac and ip_to_mac[data['ciaddr']] != data['chaddr']:
                print(f"IP {data['ciaddr']} has multiple MACs: {ip_to_mac[data['ciaddr']]} and {data['chaddr']}")
            ip_to_mac[data['ciaddr']] = data['chaddr']
        if 'opcode' in data and data['opcode'] == 'DHCPREQUEST':
            mac = data['chaddr']
            addr = data['riaddr']
            if mac not in ip_hold_time:
                ip_hold_time[mac] = []
            if ip_hold_time[mac] and ip_hold_time[mac][-1]['end'] is None:
                pass
            else:
                ip_hold_time[mac].append({
                    'start': log['_time'],
                    'end': None,
                    'addr': addr
                })
        if 'opcode' in data and data['opcode'] == 'DHCPRELEASE':
            mac = data['chaddr']
            if mac not in ip_hold_time:
                print(f"User {mac} has no start time")
            else:
                ip_hold_time[mac][-1]['end'] = log['_time']
        if 'queries' in data:
            mac = data['src_ip']
            mac = mac_to_user.get(ip_to_mac.get(mac, None), mac)
            if mac not in user_visited:
                user_visited[mac] = []
            for query in data['queries']:
                if query['name'] not in user_visited[mac]:
                    user_visited[mac].append(query['name'])
with open("ip_hold_time.json", "w") as f:
    json.dump(ip_hold_time, f, indent=4)
for mac, times in ip_hold_time.items():
    all_addr = list(set([time['addr'] for time in times]))
    if len(all_addr) > 1:
        print(f"mac {mac} has multiple addresses: {all_addr}")
        for addr in all_addr:
            print(f"{addr} -> {ip_to_name.get(addr, 'Unknown')}")

unique_visits = {}
for mac, visited in user_visited.items():
    for item in visited:
        unique_visits[item] = mac if item not in unique_visits else None

for item, mac in unique_visits.items():
    if mac is not None:
        print(f"{mac} has a unique visit to {item}")
```
